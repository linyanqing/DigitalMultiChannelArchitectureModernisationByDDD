# 08 – Delivery Roadmap & Transition Architectures

Modernisation runs incrementally using the **Strangler Fig Pattern** so **business delivery never stops**. The plan has three parts:
- **Horizon 1 (M1–6):** two 3-month planning cycles that deliver the business targets
- **Horizon 2 (M7–12):** consolidation
- **Horizon 3 (M13–24):** reaching the target state and scaling the platform

Each transition architecture (T1–T4) is a **stable, releasable state**: the programme could pause at any of them and still keep the value delivered.

| Transition | When | Theme |
|-----------|------|-------|
| **T1** | End of Cycle 1 (M3) | Platform foundation, Onboarding context & high-volume onboarding (~50% of forms) |
| **T2** | End of Cycle 2 (M6) | Profile changes & staff integration (**≥ 80%**, +30% speed) |
| **T3** | M12 | Consolidation – legacy BFFs folded in, bypasses closed, automation |
| **T4** | M24 | Target state – legacy retired, platform self-service for other domains |

## 8.1 Roadmap overview

```mermaid
gantt
  title Digital Experience & Platform Modernisation – 24 months
  dateFormat  YYYY-MM-DD
  axisFormat  %b %y
  section Cycle 1 (M1–3)
  DORA baseline + ADRs                          :c1a, 2026-10-01, 30d
  EventStorming, subdomains, context map, glossary :c1s, 2026-10-01, 21d
  Form runtime build-vs-buy evaluation          :c1v, 2026-10-01, 21d
  Onboarding context + outbox + process manager :c1o, 2026-10-15, 75d
  Unified Digital BFF core + schema registry    :c1b, 2026-10-01, 75d
  React Form Runner embedded in JSP             :c1c, 2026-10-15, 60d
  Native SDUI renderers v1 (iOS, Android)       :c1d, 2026-10-15, 75d
  Top 3 onboarding forms (web + mobile)         :c1e, 2026-11-15, 45d
  GitHub Actions templates + telemetry SDK      :c1f, 2026-10-01, 75d
  section Cycle 2 (M4–6)
  Profile Change context (event-sourced)        :c2p, 2027-01-01, 60d
  Profile change forms on SDUI                  :c2a, 2027-01-01, 80d
  Status & case read models (CQRS)              :c2q, 2027-01-15, 60d
  Retire JSP onboarding routes (reverse proxy)  :c2b, 2027-02-01, 55d
  Staff portals on BFF form APIs                :c2c, 2027-01-15, 75d
  All squads on CI/CD templates                 :c2d, 2027-01-01, 90d
  section H2 (M7–12)
  Consolidate legacy channel BFFs               :h2a, 2027-04-01, 180d
  Close domain bypasses                         :h2b, 2027-04-01, 120d
  KYC/KYB automation + STP simple merchants     :h2c, 2027-04-15, 150d
  Repatriate leaked logic (per form)            :h2d, 2027-04-01, 180d
  Outbox + OHS for existing contexts           :h2e, 2027-04-01, 150d
  section H3 (M13–24)
  Deprecate channel-specific domain APIs        :h3a, 2027-10-01, 180d
  Retire remaining in-scope JSP routes          :h3b, 2027-10-01, 270d
  Long-tail forms + self-service authoring      :h3c, 2027-10-01, 240d
  Onboard other domains (Lending, Banking)      :h3d, 2028-01-01, 270d
  section Milestones
  T1 ~50% forms                                 :milestone, m1, 2026-12-31, 0d
  T2 80% forms, +30% speed                      :milestone, m2, 2027-03-31, 0d
  T3 consolidated                               :milestone, m3, 2027-09-30, 0d
  T4 target state                               :milestone, m4, 2028-09-30, 0d
```

## 8.2 Horizon 1 – two-cycle (6-month) execution plan

### Cycle 1 (Months 1–3): Platform foundation & high-volume onboarding
0. **Weeks 1–3 – strategic design:** EventStorming on onboarding and profile change; agree core/supporting/generic subdomains, the context map and glossary v1 ([11](11-ubiquitous-language.md)); time-boxed **build-vs-buy** decision for the form runtime.
1. Deploy the **Unified Digital BFF core** (stateless) and the **JSON Schema Registry** on the adopted form runtime.
2. Build the **Merchant Onboarding bounded context**: `MerchantApplication` aggregate, process manager, **transactional outbox**, ACLs to IDV/e-sign.
3. Embed the **React Form Runner** container inside the legacy JSP web shell.
4. Digitise the **top 3 high-volume merchant onboarding** paper forms on web & mobile (**~50% of form target reached**).
5. Establish unified **CI/CD templates** (GitHub Actions) and the shared client **telemetry SDK**.

### Cycle 2 (Months 4–6): Profile changes & staff integration
1. Build the **Profile Change bounded context** (event-sourced `ProfileChangeRequest`) and convert **profile change** forms (settlement account, address, directors) to SDUI (**reaches the 80%+ conversion target**).
2. Add **CQRS read models** for application/change status and staff case views.
3. **Retire legacy JSP onboarding routes** via reverse-proxy URL strangulation.
4. **Expose the Digital BFF form APIs to staff portals** (assisted submissions, referrals), replacing re-keying for these forms.
5. Complete **cross-squad adoption** of standardised CI/CD pipelines.

> **Why onboarding first?** It is a **core subdomain** and the highest-volume form set (growth + drop-off), and it exercises every platform component (schema, BFF, process manager, outbox, documents, verification). Investing in core first follows DDD guidance. Profile changes (a supporting subdomain) then reuse a proven platform, which makes Cycle 2 faster.
>
> **De-risking week 1–6:** ship one low-risk form (e.g. contact details) as a **walking skeleton** through BFF → gateway → context on all three channels before the onboarding forms land.

## 8.3 Transition architecture T1 (M3) – "Platform foundation"

```mermaid
flowchart TB
  classDef new fill:#dcfce7,stroke:#15803d
  classDef old fill:#f3f4f6,stroke:#6b7280

  subgraph Channels
    JSP[Legacy JSP web<br/>+ embedded React Form Runner]:::new
    IOS[iOS app + SDUI renderer v1]:::new
    AND[Android app + SDUI renderer v1]:::new
    SP[Staff portals – unchanged]:::old
  end
  BFF[Unified Digital BFF core<br/>stateless]:::new
  REG[(Schema registry<br/>adopted form runtime)]:::new
  GW[API Gateway]
  ONB[Merchant Onboarding context<br/>aggregate + process manager + outbox]:::new
  ACL[ACLs: IDV / e-sign / legacy]:::new
  BUS[[Event bus]]:::new
  subgraph Legacy
    WBFF[Web BFF]:::old
    IBFF[iOS BFF]:::old
    ABFF[Android BFF]:::old
  end
  subgraph Domains
    CUS[Customer]
    BNK[Banking]
    PAY[Payments]
  end

  JSP -- "onboarding forms" --> BFF
  IOS -- "onboarding forms" --> BFF
  AND -- "onboarding forms" --> BFF
  BFF --> REG
  BFF --> GW --> ONB
  ONB --> CUS & BNK & PAY
  ONB --> ACL
  ONB --> BUS
  JSP --> WBFF
  IOS --> IBFF
  AND --> ABFF
  WBFF & IBFF & ABFF --> CUS & BNK & PAY
  SP --> CUS & BNK & PAY
```

**True at T1:** new onboarding forms run on the new stack in all three channels, and the application lifecycle lives in the **Onboarding context**, not the BFF. Cross-context steps run through the process manager with outbox events. Everything else stays on the legacy BFFs, so BAU delivery is untouched. Referrals are still handled in the existing staff portals.

## 8.4 Transition architecture T2 (M6) – "Profile changes & staff integration"

```mermaid
flowchart TB
  classDef new fill:#dcfce7,stroke:#15803d
  classDef old fill:#f3f4f6,stroke:#6b7280

  RP[Reverse proxy]:::new
  RSH[React shell – /onboarding routes]:::new
  JSP[JSP – other pages<br/>+ React Form Runner for profile changes]:::old
  IOS[iOS – SDUI]:::new
  AND[Android – SDUI]:::new
  SP[Staff portals + Form Runner<br/>staff scopes]:::new
  BFF[Unified Digital BFF – stateless<br/>onboarding · profile-change · schema modules]:::new
  GW[API Gateway]
  LBFF[Legacy BFFs – BAU screens only]:::old
  ONB[Merchant Onboarding context]:::new
  PCR[Profile Change context<br/>event-sourced]:::new
  RM[Status & case read models - CQRS]:::new
  OPS[Operations – Case]:::new
  BUS[[Event bus]]:::new
  CUS[Customer]
  BNK[Banking]
  PAY[Payments]

  RP --> RSH & JSP
  RSH & JSP & IOS & AND & SP --> BFF
  BFF --> GW --> ONB & PCR & RM
  ONB --> CUS & BNK & PAY
  PCR --> CUS & BNK
  ONB & PCR --> BUS --> RM & OPS
  JSP & IOS & AND -.-> LBFF -.-> GW
```

**True at T2:** ≥ 80% of paper volume is digital. Both new bounded contexts are live, and status is served from read models. JSP onboarding routes are retired, and staff handle the same forms (assisted or referred) through the BFF, with cases raised from events. All squads are on the paved-road pipelines.

## 8.5 Transition architecture T3 (M12) – "Consolidation"
- Legacy channel BFF endpoints for customer-profile and onboarding-adjacent screens are folded into the Unified BFF modules; remaining BFF scope is small and scheduled for removal.
- **Direct domain bypasses closed**: all client traffic goes through BFF → API Gateway.
- Leaked rules for migrated forms are repatriated into their owning bounded contexts, with parity tests.
- Existing contexts (Customer, Banking, Payments) expose **open-host services with a published language** and publish integration events via outbox.
- KYC/KYB automation and **STP for low-risk merchants** (Onboarding ↔ Risk partnership).

## 8.6 Transition architecture T4 (M24) – "Target state" (see [07](07-target-architecture.md))
- Channel-specific domain APIs (e.g. iOS-only) are **deprecated and removed**; the legacy ACLs are removed with them.
- Remaining in-scope JSP routes are retired; web runs on the React shell.
- Long-tail forms are digitised through **self-service schema authoring** with governance guardrails.
- Other domains (Lending applications, Banking account opening) adopt the BFF + Form Engine paved road, each with its own bounded context.
- Subdomain classification reviewed (e.g. if STP becomes routine, parts of risk decisioning may move from core to supporting).

## 8.7 Risks, dependencies & constraints (RAID)

| Type | Item | Mitigation |
|------|------|-----------|
| Risk | Native SDUI renderers not ready for Cycle 1 | Minimal component catalogue; walking skeleton by week 6; contingency: Form Runner in webview for a specific form |
| Risk | State or business logic creeps into the BFF or schemas | Stateless-BFF rule, fitness tests, schema = presentation only, design-authority review |
| Risk | Two new bounded contexts stretch Cycle 1–2 capacity | Onboarding (core) in C1, Profile Change in C2; lean aggregates first |
| Risk | Adopted form runtime doesn't fit native renderers | Time-boxed evaluation; adapter; build native renderers against adopted schema |
| Risk | 80% target misread (form count vs volume) | Agree volume-based measure with Product in week 1 |
| Risk | BAU demand starves platform capacity | Ring-fenced platform team; squads split new/BAU capacity |
| Risk | Old app versions can't render new schemas | Version negotiation, min-version policy, web fallback |
| Risk | Staff portals' tech diversity slows Form Runner adoption | Runner shipped as framework-agnostic web component |
| Dependency | Identity provider supports PKCE + step-up | Early engagement; OTP fallback |
| Dependency | Existing contexts accept commands (RegisterParty, NominateSettlementAccount) and publish events | Joint planning; ACL over current APIs until OHS is ready |
| Dependency | Vendors (IDV, e-sign) | Adapters; decide by week 6 |
| Constraint | Zero delivery stoppage | Strangler Fig, feature flags, legacy BFFs kept for BAU |
| Constraint | Regulatory (AML/CTF, prudential, privacy) | Compliance in design authority; audit by design |

## 8.8 Governance & exit criteria
- **Per cycle:** transition architecture review; benefits tracking (digital form %, DORA, drop-off, ops handling time).
- **Fortnightly:** design authority – ADRs, context map and glossary changes, schema/catalogue changes, BFF boundary checks.
- **T1 exit:** subdomains, context map and glossary v1 agreed; Onboarding context live with outbox; top 3 onboarding forms live on 3 channels; ~50% of paper volume digital; templates and telemetry SDK live; zero Sev1 from the new stack.
- **T2 exit:** Profile Change context and read models live; ≥ 80% of paper volume digital; JSP onboarding routes retired; staff portals on BFF APIs; −30% lead time vs baseline.
