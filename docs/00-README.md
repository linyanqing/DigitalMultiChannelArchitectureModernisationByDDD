# Digital Experience & Platform Modernisation – Architecture Case Study

**Role:** Digital Platform Architect (Web & Mobile)
**Scope:** Merchant Onboarding & In-Life Customer Profile Maintenance
**Target delivery window:** Two planning cycles (~6 months), within a 2-year roadmap
**Stakeholders:** Principal Engineer, Engineering Lead, Product Manager
**Source brief:** [problemStatement.txt](problemStatement.txt)

## Executive summary

A payments and business-banking provider's customer touchpoints have expanded organically across web and native mobile channels, each managed by an independent engineering team. This siloed evolution has produced inconsistent customer journeys, fragmented developer practices and delayed feature rollouts.

The modernisation focuses on digitising the paper-based **merchant onboarding** and **in-life profile maintenance** journeys. The target architecture establishes explicit **bounded contexts** for Merchant Onboarding and Profile Change, a stateless **Unified Digital BFF** and a **Server-Driven UI (SDUI) JSON Form Engine**, supported by a standardised **Engineering Paved Road**. Channels are modernised incrementally (Strangler Fig), so feature delivery speeds up with **zero disruption to ongoing business delivery**.

| # | Document | Use in the session |
|---|----------|--------------------|
| 01 | [Business Problem](01-business-problem.md) | Opening – align on "why" (5 min) |
| 02 | [Business Benefits](02-business-benefits.md) | Goals, measures, success criteria |
| 03 | [As-Is Business Process](03-as-is-business-process.md) | Current state walk-through with PM |
| 04 | [As-Is User Experience](04-as-is-user-experience.md) | Merchant & staff pain points |
| 05 | [As-Is Architecture](05-as-is-architecture.md) | Context / deployment / sequence – validate with Principal Eng |
| 06 | [Gap Analysis](06-gap-analysis.md) | Bridge from current to target |
| 07 | [Target Architecture](07-target-architecture.md) | Context / deployment / sequence + key design decisions |
| 08 | [Delivery Roadmap & Transition Architectures](08-delivery-roadmap.md) | 2-cycle execution plan + 2-year roadmap, T1–T4 transition states |
| 09 | [Engineering Roadmap](09-engineering-roadmap.md) | Paved road, team topology, practices aligned to T1–T4 |
| 10 | [DDD Review](10-ddd-review.md) | Review against *Learning Domain-Driven Design* (Khononov) – findings & actions (applied) |
| 11 | [Ubiquitous Language](11-ubiquitous-language.md) | Glossary per bounded context – use the same terms on the whiteboard |
| ADR | [ADR-001 Unified Digital BFF](adr/ADR-001-unified-digital-bff.md) | Why keep one stateless BFF; BFF vs gateway vs facade; pass-through test |

---

## One-slide summary (the story to tell)

> **Define a form once, render it everywhere, change it without an app release – and modernise the channels incrementally behind it, never as a pre-requisite.**

1. **Problem** – paper-based onboarding & profile maintenance; three channel silos (JSP, Swift, Kotlin) with fragmented BFFs, BFF bypassing, channel-specific domain APIs, leaked logic, and web/mobile release desynchronisation ⇒ merchant drop-off, ops overhead, slow delivery.
2. **Goal (6 months)** – 80% of paper forms become digital self-service; digital feature delivery +30%; zero delivery stoppage.
3. **Core idea** –
   - **Strategic design first**: onboarding and risk decisioning are **core** subdomains (build); profile changes and ops cases are **supporting**; the form runtime, IDV, e-sign, documents and notifications are **generic** (buy/adopt behind ACLs).
   - **Bounded contexts**: new **Merchant Onboarding** (`MerchantApplication` aggregate + process manager + outbox) and **Profile Change** (event-sourced, for audit) contexts own the lifecycles; a **context map** names the relationships (open-host services, partnership, ACLs).
   - **Unified Digital BFF (stateless)**: one channel contract for Web, iOS, Android *and Staff Portals* – auth (OAuth2/PKCE), schema serving, translating forms into commands. No state, no business rules.
   - **SDUI JSON Form Engine**: schemas carry presentation only and are owned by the context team; native SwiftUI/Compose renderers on mobile; **React Form Runners embedded in legacy JSP** on web.
   - **Core domain isolation**: contexts reachable only via the API Gateway; status served from **CQRS read models**.
   - **Engineering Paved Road**: tokenised design system, GitHub Actions templates, shared OpenTelemetry SDK.
4. **Adoption** – **Strangler Fig**, not rebuild: React runners inside JSP first, then reverse-proxy route retirement; legacy BFFs keep serving BAU until consolidated.
5. **Sequencing** – **Cycle 1 (M1–3)**: EventStorming + context map (weeks 1–3), Onboarding context + outbox, BFF core + schema registry, React runner in JSP, top 3 onboarding forms on web & mobile (~50%) → **Cycle 2 (M4–6)**: Profile Change context + profile change forms (80%+), read models, retire JSP onboarding routes, staff portals on BFF form APIs → **H2 (M7–12)**: BFF consolidation, logic repatriation, automation → **H3 (M13–24)**: target state, legacy retired, platform self-service for other domains.

---

## Key assumptions (state these up front, validate in the room)

| # | Assumption | Why it matters |
|---|-----------|----------------|
| A1 | A "planning cycle" is a quarter; two cycles ≈ 6 months; 2-year roadmap ≈ 8 cycles | Roadmap granularity |
| A2 | "80% of paper forms" is measured by **submission volume**, not form count (confirm!) | Changes prioritisation completely – Pareto by volume |
| A3 | ~30–60 paper/PDF forms exist across onboarding + maintenance (e.g. merchant application, settlement account change, add/remove director/signatory, trading name/address change, additional terminal/facility, ownership change, closure) | Sizing of form engine vs bespoke build |
| A4 | Fulfilment today is manual (ops staff re-key into domain staff portals / back-office) | Staff portals reusing the BFF form APIs removes re-keying early |
| A5 | Backend is cloud-hosted Kubernetes with API gateway; Kafka or similar event backbone exists or is planned (confirm AWS/GCP) | Deployment diagrams |
| A6 | Regulatory: AML/CTF (AUSTRAC) KYC/KYB, Privacy Act, APRA CPS 234/230, PCI DSS – apply to onboarding & profile changes | Identity verification, audit, data residency |
| A7 | Mobile apps have an authenticated merchant session; new-merchant onboarding may start unauthenticated (web first) | Journey entry points |
| A8 | Current delivery baseline (lead time, release frequency) is not consistently measured | Need to baseline before claiming +30% |

## Clarifying questions to ask the panel

**Product Manager**
- How is "80% of paper forms" measured – count of forms or volume of submissions? Which 5–10 forms carry most volume?
- Which parts of onboarding differentiate us from competitors (speed, approval rate)? → confirms the **core subdomain**
- Which journeys *must* differ by channel (e.g. mobile camera ID capture, tap-to-pay, push)? Where is parity a hard requirement?
- Is new-merchant onboarding expected in-app (app download before being a customer) or web-first?
- What is the current drop-off / turnaround time for onboarding (days to first transaction)?

**Principal Engineer**
- Which BFFs exist, who owns them, and which domain APIs are channel-specific (iOS)? Where does leaked logic live?
- Is there an existing workflow/case management capability, document store, e-signature, IDV vendor, event bus / outbox practice?
- Do domain teams already publish events or versioned public APIs (open-host services)?
- Any enterprise standards already chosen (API gateway, event bus, IdP, design system, micro-frontend approach)?
- Constraints on JSP app – can we route by path at the edge / reverse-proxy?

**Engineering Lead**
- Team shape and size per channel? Appetite for cross-channel (feature) teams vs channel teams?
- Mobile release cadence and app-store lead time? Existing feature-flag / remote-config tooling?
- Current CI/CD and test automation maturity per channel; DORA metrics available?

## 90-minute workshop plan (you drive, keep time)

| Time | Segment | Output on whiteboard |
|------|---------|----------------------|
| 0–10 | Intros, objectives, agenda, assumptions | Goals + measures (80%, +30%, no freeze) |
| 10–30 | **Big-picture EventStorming**: "applicant applies → first transaction" and "merchant changes settlement account"; mark hot spots; validate as-is context diagram | Event timeline, hot spots, candidate aggregates & context boundaries |
| 30–38 | Problem framing, subdomains (core/supporting/generic), principles | Subdomain map, principles |
| 38–60 | Target architecture: bounded contexts + context map, stateless BFF, form engine build-vs-buy, key decisions | Context map, target context diagram, decision log |
| 60–75 | Adoption / sequencing – T1..T4, first lighthouse journey | Roadmap with transition states |
| 75–85 | Risks, dependencies, constraints, team topology | RAID list |
| 85–90 | Recap, open questions, next steps | Summary + actions |
