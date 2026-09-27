# 03 – As-Is Business Process

> Reconstructed from the brief plus typical merchant-acquirer practice. **Validate with the Product Manager in the session** – the point is to align on the shape of the process and where the waste is, not exact steps.

## Actors
| Actor | Role |
|-------|------|
| Merchant (applicant / existing customer) | Business owner, director, authorised signatory |
| Sales / Partner (e.g. POS integration partner, BDM) | Originates leads, helps merchants complete forms |
| Onboarding Ops | Receives forms, re-keys data, chases missing info |
| Risk & Compliance (KYC/KYB, credit) | AML/CTF verification, credit/fraud assessment, approval |
| Customer Service / Servicing Ops | Processes in-life change requests |
| Domain systems | Customer, Banking, Lending, Acquiring (merchant/terminal), CRM |
| Logistics | Terminal provisioning and dispatch |

---

## Process 1 – New merchant onboarding (as-is)

```mermaid
flowchart TD
  A([Merchant enquiry<br/>web / sales / partner]) --> B[Download PDF application<br/>or receive from BDM]
  B --> C[Merchant completes form by hand<br/>+ gathers ID, ABN, bank statements]
  C --> D[Sign wet-ink / scan / email or post]
  D --> E[Onboarding Ops inbox]
  E --> F{Complete &<br/>in good order?}
  F -- No / NIGO --> G[Ops chases merchant<br/>email / phone]
  G --> C
  F -- Yes --> H[Ops re-keys data into<br/>staff portals per domain]
  H --> I[Manual/semi-manual KYC/KYB<br/>ABN lookup, ID check, beneficial owners]
  I --> J[Credit & risk assessment]
  J --> K{Approved?}
  K -- Refer --> G
  K -- Decline --> L([Decline letter/email])
  K -- Approve --> M[Create customer + merchant<br/>+ settlement account setup]
  M --> N[Order & configure terminal]
  N --> O[Dispatch / activation]
  O --> P([Welcome pack; merchant<br/>downloads app / logs in to web])
```

### Observations / waste
| # | Step | Waste (Lean) | Impact |
|---|------|-------------|--------|
| P1 | Paper form | Defects (illegible, incomplete) | High NIGO rate, rework loops |
| P2 | Email/post | Waiting, transport | Days of latency; no status visibility |
| P3 | Re-keying | Over-processing, defects | Ops cost; data errors |
| P4 | Manual KYC | Over-processing | Inconsistent decisions, slow |
| P5 | Chase loops | Waiting, motion | Abandonment; poor NPS |
| P6 | Multiple staff portals | Motion (swivel-chair) | Long handling time |
| P7 | No single case record | Information silos | No end-to-end SLA tracking, poor auditability |

---

## Process 2 – In-life profile maintenance (as-is)

Typical requests: change of business/trading name, address, contact details, **settlement bank account**, add/remove director or authorised signatory, change of ownership, add product/terminal, update MCC/business activity, close facility.

```mermaid
flowchart TD
  A([Merchant needs to change details]) --> B{Available digitally<br/>in their channel?}
  B -- Yes, some simple fields<br/>e.g. contact on web --> C[Self-service in web JSP<br/>or app – differs by channel]
  C --> Z([Updated – channel-specific rules])
  B -- No --> D[Call support or<br/>download PDF form]
  D --> E[Complete, sign, attach docs<br/>e.g. bank statement, ID]
  E --> F[Email / upload / post]
  F --> G[Servicing Ops queue]
  G --> H{Verification needed?<br/>e.g. bank acct change,<br/>new signatory}
  H -- Yes --> I[Call-back / ID check / fraud check]
  I --> J{OK?}
  J -- No --> K[Chase merchant] --> E
  J -- Yes --> L
  H -- No --> L[Re-key in domain staff portal<br/>Customer / Banking / Acquiring]
  L --> M[Manual confirmation email]
  M --> N([Change effective<br/>often days later])
```

### Observations
- **Channel inconsistency**: the same change may be self-service on web, unavailable in iOS, and paper-only in Android.
- **High-risk changes** (settlement account, signatories) are paper-based partly *because* the digital channels lack step-up authentication and verification – a platform gap, not a product decision.
- No status tracking; merchants call support to check progress ⇒ extra contact volume.
- Rules for what needs verification live in ops procedures and (partially) in channel code – inconsistent.

---

## Domain events view (EventStorming input)

The same processes, written as **domain events** (past tense, business language). Run this as a big-picture EventStorming in the workshop; hot spots (🔥) mark pain and missing ownership.

```mermaid
flowchart LR
  classDef ev fill:#fdba74,stroke:#c2410c,color:#000
  classDef hot fill:#f9a8d4,stroke:#be185d,color:#000
  classDef agg fill:#fde047,stroke:#a16207,color:#000

  E1[Application form requested]:::ev --> E2[Application received]:::ev
  E2 --> H1[🔥 Application not in good order]:::hot
  H1 --> E3[Missing information requested]:::ev --> E2
  E2 --> E4[Application keyed into systems]:::ev
  E4 --> E5[Identity verified]:::ev --> E6[Beneficial owners verified]:::ev
  E6 --> E7[Risk assessed]:::ev
  E7 --> E8[Application approved]:::ev
  E7 --> E9[Application referred]:::ev
  E7 --> E10[Application declined]:::ev
  E8 --> E11[Party registered]:::ev --> E12[Settlement account nominated]:::ev
  E12 --> E13[Merchant onboarded]:::ev --> E14[Terminal dispatched]:::ev --> E15[First transaction processed]:::ev
  H2[🔥 Nobody owns the application end-to-end]:::hot -.-> E4
  A1[Aggregate candidate:<br/>MerchantApplication]:::agg -.-> E2
```

| Profile change events | Owning context (target) |
|-----------------------|-------------------------|
| Change requested → Signatory verified → Change approved → Settlement account changed → Merchant notified | Profile Change (request lifecycle), Banking (settlement account) |
| Contact details changed | Customer / Party |
| Director added → Director verified | Profile Change → Customer / Party |

Hot spots found: re-keying (no owning model), NIGO loops (no invariants at capture), call-back verification (no step-up), status enquiries (no read model).

## Volume & value hypothesis (to validate)

| Form / request | Est. share of paper volume | Complexity | Candidate for |
|----------------|---------------------------|-----------|---------------|
| Merchant application (sole trader / company) | 25–35% | High (KYC/KYB, credit) | **Cycle 1** – top 3 onboarding forms; STP later (H2) |
| Settlement bank account change | 10–15% | Medium (fraud risk, step-up) | **Cycle 2** |
| Contact / address / trading name change | 15–20% | Low | **Cycle 2** (quick win; can be walking skeleton in Cycle 1) |
| Add/remove director / signatory | 5–10% | Medium–High (KYC) | **Cycle 2** |
| Additional terminal / product request | 10–15% | Low–Medium | Cycle 1 (onboarding add-on) or Cycle 2 |
| Change of ownership | < 5% | High | H2 / staff-assisted via staff portal on BFF form APIs |
| Closure / cancellation | < 5% | Medium (retention) | H2 |

> **Speaking point:** apply Pareto – typically ~6–8 forms make up ~80% of volume. Digitise those first, using a config-driven form engine so the long tail becomes cheap.
