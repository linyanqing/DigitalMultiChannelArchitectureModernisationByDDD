# 01 – Business Problem

## Problem statement

> The company's merchant customer experience has grown organically across web, iOS and Android, each owned by a separate team with its own stack, BFF and release rhythm. Key merchant journeys – **new customer onboarding** and **in-life profile maintenance** – still rely heavily on **paper/PDF forms** that are manually processed by operations staff. The result is slow, error-prone and inconsistent customer journeys, high cost-to-serve, and slow, expensive feature delivery that cannot keep pace with the business.

## Business domain & subdomains

**Business domain:** payments and business banking for small and medium merchants.

| Subdomain | Type | Relevance to this problem |
|-----------|------|---------------------------|
| Merchant onboarding | **Core** | Time-to-first-transaction and approval rate are competitive levers, and today they are paper-bound |
| Risk decisioning | **Core** | Needed for straight-through processing |
| Payments / acquiring | **Core** | Where onboarded merchants create value |
| Profile change requests | **Supporting** | Necessary, but doesn't differentiate; audit-heavy |
| Ops case handling | **Supporting** | Referrals and exceptions |
| Forms, IDV, e-signature, documents, notifications | **Generic** | Solved problems – buy/adopt |

Terms follow the [ubiquitous language](11-ubiquitous-language.md).

## The four headline problems

| # | Problem | What it means |
|---|---------|---------------|
| 1 | **Manual & paper-based friction** | Onboarding and profile maintenance rely on paper forms, increasing merchant drop-off and operational overhead |
| 2 | **Channel inconsistency** | Disjointed journeys across Web, iOS and Android fragment the brand and the experience |
| 3 | **Delivery drag** | The same feature is engineered three times across three stacks (JSP, Swift, Kotlin) |
| 4 | **Release desynchronisation** | Fast web deployments outpace mobile app-store release trains, creating feature-parity gaps |

## Problem breakdown

### 1. Customer (merchant) problems
| Problem | Evidence / symptom |
|---------|-------------------|
| Paper-based journeys | Merchants download/print PDFs, sign, scan and email/post; no progress visibility |
| Slow time-to-value | Onboarding takes days/weeks due to re-keying, missing info and back-and-forth ("not in good order" – NIGO) |
| Inconsistent experience | Same task looks and behaves differently (or is unavailable) across web, iOS, Android |
| Channel dead-ends | Merchants start a task in the app but are sent to web or to a paper form to finish |
| Repeated data entry | Data the company already holds is re-requested (no pre-fill) |

### 2. Operational problems
| Problem | Impact |
|---------|--------|
| Manual re-keying of form data into staff portals | Cost-to-serve, errors, compliance risk |
| Fragmented staff-facing tools per domain | Staff swivel-chair across systems; long handling time |
| Poor data quality from paper | Rework, failed KYC, delayed settlement setup |
| Limited audit trail/traceability for changes | Compliance exposure (AML/CTF, privacy) |

### 3. Delivery / technology problems
| Problem | Impact |
|---------|--------|
| Three channel silos with different stacks, patterns, testing, observability | Each feature is built 3×; skills not transferable |
| Inconsistent integration: some channels bypass BFF, some domain APIs are channel-specific (iOS) | Tight coupling, duplicated logic, fragile change |
| Business logic leaked into channel/BFF layers | Rules diverge between channels ⇒ inconsistent outcomes; hard to change |
| Web releases more often than mobile | Feature-parity gaps; release coordination overhead |
| Legacy JSP server-side rendering | Hard to deliver modern, responsive, accessible UX; slow to change |
| Fragmented staff-facing channels | Duplicate effort; hard to add ops support for new digital journeys |

## Root causes (5-whys summary)
1. **Organisation mirrors channels, not journeys** (Conway's law) → each channel builds its own version of every journey.
2. **No shared digital platform / paved road** → every team solves auth, forms, docs, observability, release differently.
3. **No bounded contexts and no context map** → the application and change-request lifecycles have no owning model, so logic leaks into channels and BFFs; without open-host services, channels bypass layers and domains grow channel-specific APIs.
4. **No shared language** → merchant/customer/applicant/party and form/application/case are used interchangeably, so requirements and code drift apart.
5. **Journeys are code, not configuration** → every form change needs code change × 3 channels and an app-store release.
6. **Back office designed around paper** → digital channels have nothing to "plug into" except email/ops queues.

## Constraints
- **Time**: meaningful progress within **two planning cycles (~6 months)**.
- **No delivery freeze**: business-as-usual features must keep shipping.
- **No big-bang rebuild**: incremental modernisation (strangler fig).
- **Enterprise direction**: domain-aligned platformisation and reusable APIs.
- **Product flexibility**: consistent journeys, but room for channel-specific experiences.
- **Regulatory**: AML/CTF KYC/KYB, Privacy Act, APRA CPS 234/230, PCI DSS; auditability of customer data changes.
- **People**: engineers are channel-specialised; capacity is finite and shared with BAU.

## In scope / out of scope
| In scope | Out of scope (for now) |
|----------|-----------------------|
| New merchant onboarding (web + mobile) | Full rebuild of JSP web or native apps |
| In-life profile maintenance (business details, contacts, directors/signatories, settlement accounts, addresses, add-on products/terminals requests) | Payments/lending/banking transactional journeys (future adopters of the platform) |
| Staff handling of digital submissions (work queue) | Replacing all domain staff portals |
| Paved road for cross-channel journey delivery | Core domain system replacement |
