# 11 – Ubiquitous Language (Glossary)

Each term belongs to a **bounded context** (see [07 §7.2](07-target-architecture.md)). The same word can mean different things in different contexts, and that is expected. The same concept must never have two names **within** a context. Schemas, API names, events, UI copy and code all use these terms.

## Strategic terms

| Term | Meaning in this pack |
|------|---------------------|
| **Business domain** | The company's overall area of activity: payments and business banking for small and medium merchants |
| **Subdomain** | A business capability area, classified as **core** (differentiating), **supporting** (necessary, not differentiating) or **generic** (solved problem, buy/adopt) |
| **Bounded context** | A boundary within which one model and one language are consistent; owned by exactly one team |
| **Domain service / context service** | The deployable implementing a bounded context (e.g. Onboarding service) |
| **Channel** | A way a user reaches us: Web, iOS, Android, Staff portal |

## Merchant Onboarding context

| Term | Definition | Not to be confused with |
|------|-----------|-------------------------|
| **Applicant** | A business applying to become a merchant, before approval | Merchant (after approval), Party |
| **Merchant Application** | The business object (aggregate) representing one application and its lifecycle | "Form" (the UI), "Case" (ops work item) |
| **Application status** | Draft → Submitted → Verifying → Referred → Approved / Declined / Withdrawn | Journey step |
| **Director** | A person legally appointed as a director of the applicant entity | Signatory, Beneficial Owner |
| **Beneficial Owner** | A person who ultimately owns or controls ≥ 25% of the entity (AML/CTF) | Director |
| **Authorised Signatory** | A person authorised to sign on behalf of the business | Director |
| **Supporting Document** | Evidence attached to an application (ID, trust deed, bank statement) | Form |
| **Referral** | An application that needs human review before a decision | Decline |

## Profile Change context

| Term | Definition | Not to be confused with |
|------|-----------|-------------------------|
| **Profile Change Request** | A request by a merchant (or staff on their behalf) to change registered details; the aggregate of this context | "Profile maintenance", "servicing" (retired synonyms) |
| **Change type** | Contact details, Registered address, Trading name, Settlement account, Signatory, Director, Ownership | – |
| **Dual authorisation** | Rule (published by Customer context) that a change needs a second signatory's approval | Step-up authentication |
| **Effective date** | When an approved change takes effect in the owning context | Submission date |

## Customer / Party context

| Term | Definition |
|------|-----------|
| **Party** | A person or organisation known to the company (legal entity or individual) |
| **Relationship** | A role one Party plays for another (director of, signatory for, beneficial owner of) |
| **Customer** | A Party holding at least one product |

## Banking context

| Term | Definition |
|------|-----------|
| **Settlement Account** | The bank account into which card-payment proceeds are settled (not "bank account" in general) |
| **Nomination** | Registering a settlement account for a merchant, pending verification |

## Payments / Acquiring context

| Term | Definition |
|------|-----------|
| **Merchant** | An approved business able to accept payments (created from an approved Merchant Application) |
| **Merchant facility** | The acquiring product/configuration for a merchant |
| **Terminal** | A payment device allocated to a merchant facility |

## Operations context

| Term | Definition |
|------|-----------|
| **Case** | A work item for staff (e.g. a referral, a failed step needing attention), linked to an application or change request |
| **Assisted submission** | A form completed by staff on behalf of a merchant ("on behalf of" audit applies) |

## Digital experience (supporting / generic)

| Term | Definition |
|------|-----------|
| **Form** | The UI definition (JSON schema) that captures input for a command – owned by the bounded context whose command it feeds |
| **Renderer** | Channel component that displays a form (React Form Runner, SwiftUI renderer, Compose renderer) |
| **Journey** | The end-to-end user experience across one or more forms and screens |
| **Command** | An intention sent to a bounded context, e.g. `SubmitApplication`, `RequestSettlementAccountChange` |
| **Domain event** | A fact inside a context, e.g. `ApplicationSubmitted` (private unless published) |
| **Integration event** | A public event in the published language, e.g. `MerchantOnboarded`, `SettlementAccountChanged` |

## Retired / ambiguous terms

| Avoid | Use instead |
|-------|------------|
| "customer" for someone applying | **Applicant** |
| "profile maintenance", "servicing", "in-life change" | **Profile Change Request** |
| "bank account" | **Settlement Account** (or specify) |
| "domain" for a service or team | **Bounded context** (model) / **subdomain** (business area) |
| "case" for an application | **Merchant Application**; "case" = ops work item only |
