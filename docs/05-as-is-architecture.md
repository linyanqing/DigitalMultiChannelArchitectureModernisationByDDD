# 05 – As-Is Architecture

> Based on the brief; hosting details are assumptions (cloud Kubernetes, API gateway). **Validate with the Principal Engineer** – especially which channels bypass the BFF and where logic has leaked.

## 5.1 Context diagram (as-is)

```mermaid
flowchart TB
  classDef person fill:#08427b,color:#fff,stroke:#052e56
  classDef sys fill:#1168bd,color:#fff,stroke:#0b4884
  classDef ext fill:#999,color:#fff,stroke:#6b6b6b
  classDef pain fill:#fde2e2,stroke:#c0392b,color:#7b1d1d

  M([Merchant / Applicant]):::person
  S([Staff<br/>Onboarding, Servicing, Risk]):::person
  P([Partner / BDM]):::person

  subgraph Channels["Customer-facing channels (separate teams)"]
    WEB[Merchant Web Portal<br/>JSP server-side rendered]:::sys
    IOS[iOS App<br/>Swift]:::sys
    AND[Android App<br/>Kotlin]:::sys
    PDF[Paper / PDF forms<br/>email · post]:::pain
  end

  subgraph ChannelLayer["Channel orchestration (per channel)"]
    WBFF[Web orchestration / BFF]:::sys
    IBFF[iOS BFF]:::sys
    ABFF[Android BFF]:::sys
  end

  subgraph Domains["Domain micro-services (Java/Spring)"]
    CUS[Customer API]:::sys
    BNK[Banking API]:::sys
    LND[Lending API]:::sys
    ACQ[Payments / Acquiring API<br/>merchant, terminals]:::sys
    IOSAPI[iOS-specific domain APIs]:::pain
  end

  subgraph Staff["Staff-facing channels (per domain, fragmented stacks)"]
    SCUS[Customer staff portal]:::sys
    SBNK[Banking staff portal]:::sys
    SACQ[Acquiring staff portal]:::sys
    INBOX[Shared ops inbox / email]:::pain
  end

  subgraph Ext["External services"]
    IDV[KYC / IDV / credit bureau]:::ext
    ABR[ABN Lookup - ABR]:::ext
    TMS[Terminal logistics]:::ext
    CRM[CRM]:::ext
  end

  M --> WEB & IOS & AND & PDF
  P --> PDF
  WEB --> WBFF
  IOS --> IBFF
  AND --> ABFF
  WBFF --> CUS & BNK & ACQ
  IBFF --> CUS & ACQ & IOSAPI
  ABFF --> CUS & LND
  AND -. "bypass: direct domain calls" .-> BNK
  WEB -. "bypass" .-> ACQ
  PDF --> INBOX
  S --> INBOX & SCUS & SBNK & SACQ
  SCUS --> CUS
  SBNK --> BNK
  SACQ --> ACQ
  CUS -.-> IDV & ABR & CRM
  ACQ -.-> TMS
```

**Pain annotations (write these on the whiteboard):**
1. Three parallel stacks → every journey built 3×.
2. Dashed "bypass" lines → no single contract, security & change coupling.
3. iOS-specific domain APIs → domain layer polluted by channel concerns.
4. Business logic in JSP / BFF / app code → divergent rules (e.g. validation, eligibility).
5. Paper → inbox → re-key → staff portal → domain. Digital channels don't reach most processes.
6. Staff portals per domain → no case view, no journey-level workflow.

## 5.2 Logical layer view – where logic sits today

| Layer | Web | iOS | Android | Issue |
|-------|-----|-----|---------|-------|
| Presentation | JSP pages + jQuery | Swift/UIKit/SwiftUI | Kotlin/Views/Compose | 3 UI codebases, 3 design systems |
| Client logic | Validation, eligibility in JSP/JS | Validation in app | Validation in app | Rules duplicated & divergent |
| Channel orchestration | Web BFF (some pages bypass) | iOS BFF + iOS-specific APIs | Android BFF (partial, some direct calls) | Inconsistent, channel-coupled |
| Domain | Customer, Banking, Lending, Acquiring | ← same | ← same | Some channel-specific endpoints; some logic missing (lives upstream) |
| Process | Email/inbox, ops procedures | – | – | No workflow/case management |

### Architectural pain points
1. **BFF bypassing** – client apps call backend domain services directly, coupling front ends to core systems.
2. **Channel-specific domain APIs** – core Java microservices host endpoints tailored to one client (e.g. iOS-only APIs).
3. **Logic leakage** – validation, orchestration and business rules are embedded in JSP pages and mobile views.

### The same pain points through a DDD lens (context-map problems)

| Symptom | DDD diagnosis |
|---------|---------------|
| Channel-specific domain APIs (iOS-only) | Domains lack an **open-host service with a published language**, so each consumer gets a bespoke contract |
| Logic leaked into JSP, apps and BFFs | **No bounded context owns** the application or change-request lifecycle, so the rules land wherever the code happens to be |
| BFF bypassing | No agreed **context map**; integration relationships are implicit and ungoverned |
| Divergent validation between channels | No **ubiquitous language** or single model; each channel has its own interpretation |
| Paper → inbox → re-key | The core onboarding subdomain has **no software model at all**; it lives in people and email |

### Engineering practice gaps
1. **Tooling silos** – Web, iOS and Android squads use different build, test, release and observability toolchains.
2. **Uncoordinated releases** – independent release cadences cause feature-parity gaps between web and mobile.
3. **Duplicated effort** – teams build the same form validation and UI layouts from scratch in three codebases.

## 5.3 Deployment diagram (as-is, assumed)

```mermaid
flowchart TB
  classDef node fill:#f5f5f5,stroke:#333
  classDef pain fill:#fde2e2,stroke:#c0392b

  subgraph Devices
    BR[Browser]
    IPH[iPhone – App Store build<br/>release ~monthly]
    ANP[Android – Play Store build<br/>release ~monthly, different day]
  end

  subgraph Edge["Edge"]
    CDN[CDN / WAF]
    GW[API Gateway<br/>per-channel routes]
  end

  subgraph Cloud["Cloud / DC – Kubernetes (assumed)"]
    subgraph NSweb["ns: web"]
      JSP[JSP web app<br/>Tomcat/Spring MVC – stateful sessions]:::pain
      WBFF[Web BFF]
    end
    subgraph NSmob["ns: mobile"]
      IBFF[iOS BFF]
      ABFF[Android BFF]
    end
    subgraph NSdom["ns: domains"]
      CUS[Customer svc]
      BNK[Banking svc]
      LND[Lending svc]
      ACQ[Acquiring svc]
    end
    subgraph NSstaff["ns: staff apps"]
      SP1[Staff portal A – e.g. Angular]
      SP2[Staff portal B – e.g. JSP]
      SP3[Staff portal C – e.g. React]
    end
    DB[(Domain DBs – per service)]
    MQ[[Messaging – partial / point-to-point]]
  end

  subgraph Obs["Observability – per team"]
    O1[Web: logs + APM tool X]
    O2[iOS: crash tool Y]
    O3[Android: crash tool Z]
  end

  subgraph CICD["CI/CD – per channel"]
    C1[Web pipeline]
    C2[iOS pipeline – Xcode Cloud / Fastlane]
    C3[Android pipeline – Gradle / Fastlane]
  end

  BR --> CDN --> JSP
  JSP --> WBFF
  IPH --> GW --> IBFF
  ANP --> GW --> ABFF
  ANP -. bypass .-> GW
  WBFF & IBFF & ABFF --> CUS & BNK & LND & ACQ
  CUS & BNK & LND & ACQ --> DB
  SP1 & SP2 & SP3 --> CUS & BNK & ACQ
```

## 5.4 Sequence diagrams (as-is)

### (a) New merchant onboarding – paper

```mermaid
sequenceDiagram
  autonumber
  actor M as Merchant
  participant W as Web (JSP)
  participant E as Ops Inbox (email)
  actor O as Onboarding Ops
  participant SP as Staff Portals (per domain)
  participant C as Customer API
  participant K as KYC/IDV vendor
  participant A as Acquiring API
  participant B as Banking API

  M->>W: Browse "Get started"
  W-->>M: PDF application form
  M->>M: Print, complete, sign, scan
  M->>E: Email form + ID + bank statement
  O->>E: Pick up email (hours–days later)
  O->>O: Completeness check
  alt Not in good order
    O->>M: Email/phone for missing info
    M->>E: Re-send documents
  end
  O->>SP: Re-key applicant data
  SP->>C: Create prospect/customer
  O->>K: Manual ID / KYB check (vendor portal)
  K-->>O: Result
  O->>SP: Record KYC outcome, credit decision
  SP->>B: Create settlement account link
  SP->>A: Create merchant + order terminal
  O->>M: Approval email (days–weeks)
```

### (b) Update business contact details – channel inconsistency

```mermaid
sequenceDiagram
  autonumber
  actor M as Merchant
  participant W as Web (JSP)
  participant WB as Web BFF
  participant I as iOS App
  participant IB as iOS BFF
  participant AN as Android App
  participant C as Customer API
  participant IA as iOS-specific Customer API

  Note over M,C: Web path
  M->>W: Edit contact details
  W->>W: Validate (JSP rules v1)
  W->>WB: POST /profile
  WB->>WB: Business rule: requires re-verify email? (logic leak)
  WB->>C: PUT /customers/{id}/contacts
  C-->>WB: 200
  WB-->>W: OK

  Note over M,IA: iOS path – different rules, different API
  M->>I: Edit contact details
  I->>I: Validate (Swift rules v2)
  I->>IB: POST /ios/profile
  IB->>IA: PUT /ios/customers/{id}
  IA->>C: update (partial fields)
  IA-->>IB: 200

  Note over M,AN: Android path – not available
  M->>AN: Look for "edit details"
  AN-->>M: "Please contact support / use web"
```

### (c) Change settlement bank account – paper with call-back

```mermaid
sequenceDiagram
  autonumber
  actor M as Merchant
  participant E as Ops Inbox
  actor O as Servicing Ops
  participant SP as Banking Staff Portal
  participant B as Banking API
  participant A as Acquiring API

  M->>E: Signed PDF + bank statement
  O->>E: Triage
  O->>M: Call-back to verify identity
  O->>SP: Re-key new BSB/account
  SP->>B: Update settlement account
  B->>A: (sync call / batch) update settlement instructions
  O->>M: Confirmation email
```

## 5.5 As-is architecture assessment

| Quality attribute | Rating | Notes |
|------------------|--------|-------|
| Consistency (cross-channel) | 🔴 | Rules and APIs differ per channel |
| Changeability | 🔴 | Change × 3 channels + app release; leaked logic |
| Reuse | 🔴 | Channel-specific APIs, no shared components |
| Security | 🟠 | Bypass paths; inconsistent auth/step-up |
| Observability | 🟠 | Per-team tools; no end-to-end journey trace |
| Operability | 🟠 | Different pipelines, practices, on-call |
| Scalability / resilience | 🟢/🟠 | Domain services OK; JSP stateful sessions limit scaling |
| Compliance / audit | 🟠 | Paper & email trail; manual evidence |
