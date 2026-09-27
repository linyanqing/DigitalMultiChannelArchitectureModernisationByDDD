# ADR-001: Keep a single, stateless Unified Digital BFF as the channel contract

| | |
|---|---|
| **Status** | Proposed – for ratification at design authority |
| **Date** | 2026-09-27 |
| **Deciders** | Digital Platform Architect, Principal Engineer, Engineering Lead, Product Manager |
| **Related** | [07 Target Architecture](../07-target-architecture.md) §7.3–7.4, D4–D6; [10 DDD Review](../10-ddd-review.md) F2–F3; [11 Ubiquitous Language](../11-ubiquitous-language.md) |

## Context

- Web (JSP), iOS and Android each have their own orchestration/BFF layer. These have drifted apart:
  - different error models, validation, telemetry and security
  - some channels bypass the BFF and call domain APIs directly
  - some domain APIs are channel-specific (iOS-only)
  - business logic has leaked into BFFs and clients
- Following the DDD review, business logic and state now live in the **Merchant Onboarding** and **Profile Change** bounded contexts, which own the aggregates, process manager and outbox. That raises the question of whether a BFF is still needed at all, and whether it is just an API gateway.
- Forces:
  - forms change over the air weekly, while context contracts should stay stable
  - mobile must work with older app versions
  - staff portals need the same forms with staff permissions
  - delivery must not stop while legacy is strangled
  - the target is +30% delivery speed

## Decision

We keep **one Unified Digital BFF** as the only entry point for digital channels (Web, iOS, Android, staff portals). It is **stateless and holds no business rules**. It sits **between an edge gateway and an internal API gateway**:

```
Devices → Edge gateway / WAF → Unified Digital BFF → Internal API gateway → Bounded contexts
```

### Responsibilities – the BFF does

| # | Responsibility | Why it belongs here |
|---|----------------|--------------------|
| R1 | Channel security: token handling (OAuth2/PKCE), token exchange for JSP sessions, triggering step-up, staff "on behalf of" scopes and audit | One hardened public boundary; contexts stay internal and trust the gateway |
| R2 | Schema serving and renderer version negotiation, feature-flag cohorts, web fallback | App versions and UI rollout are channel concerns and must not leak into the core domain |
| R3 | Translation: form payload → command (`SubmitApplication`); query results → renderer-shaped responses | Absorbs weekly form changes so context contracts stay stable |
| R4 | Read composition for screens (pre-fill from Customer, Banking and the status read model in one call) | One mobile round trip instead of 3–4; composition written once, not ×3 |
| R5 | Stable contract over a changing backend (legacy ACLs today, open-host services tomorrow) | Enables the Strangler Fig without app releases |
| R6 | Uniform error model, correlation IDs, telemetry, localisation keys | Removes the drift that caused today's inconsistency |

### Constraints – the BFF must not

- hold state: drafts are the `Draft` state of the aggregate; no BFF database
- orchestrate commands across contexts (that is the process manager's job)
- implement business rules (eligibility, dual authorisation, risk thresholds)
- expose channel-specific endpoints (channel differences go in schema hints)

### Placement relative to gateways

| Concern | Edge gateway / WAF | Unified Digital BFF | Internal API gateway |
|---------|-------------------|---------------------|----------------------|
| TLS, WAF, bot protection, DDoS | ✅ | | |
| Token signature/expiry, rate limits, quotas | ✅ | | ✅ (service calls) |
| Understands forms, screens, commands | | ✅ | |
| Fine-grained channel auth (step-up, staff scopes) | | ✅ | |
| Routing to contexts, policy, mTLS | | | ✅ |
| Owned by | Platform / infra | Context teams (modules) + platform (core) | Platform / infra |

## Pattern lineage

| Pattern | Source | How it applies |
|---------|--------|----------------|
| **Backends for Frontends** | Sam Newman | The architectural pattern. We deliberately run one BFF across channels instead of one per client; channel differences go in schema hints |
| **Facade Controller** | Larman, GRASP | Receives "system operations" from every UI and delegates to the owning context. Larman's "bloated controller" warning = our logic-creep risk; his "use-case controllers" fix = modules per bounded context |
| **Remote Facade** + DTO | Fowler, *PoEAA* | Coarse-grained remote interface over fine-grained services; fewer round trips (R4) |
| **Adapter / translation layer** | GoF; DDD | Form ↔ command translation, like an anticorruption layer in reverse: it protects the contexts' published language from UI churn (R3) |
| **API Composition** | Richardson, *Microservices Patterns* | Read-side composition for screens (R4). Note: Richardson's "API Gateway pattern" is an *application-level* entry point, of which BFF is a variant – not the infrastructure gateway products in the table above |

Where the analogy stops: Facade and Controller are in-process object patterns. The BFF is a separately deployed, network-facing service, so it also carries security, version negotiation, resilience and its own release cadence.

## Options considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **A. No BFF – renderers call contexts via the gateway** | Fewest hops; simplest topology | R1–R6 re-implemented in React, Swift and Kotlin (×3 again); app versions and UI concerns leak into contexts; every backend change forces app releases | Rejected |
| **B. One BFF per channel** (classic BFF) | Each channel optimised; team autonomy | Recreates today's divergence, channel-specific APIs and triplicated work | Rejected |
| **C. Push BFF logic into the API gateway** (plugins, mapping templates) | No extra service | "Overambitious API gateway" anti-pattern (ThoughtWorks Tech Radar – Hold): hard to test and version, owned by infra, vendor lock-in | Rejected |
| **D. GraphQL federation / supergraph** | Flexible reads; strong schema tooling | Heavy platform investment for Cycle 1; mutations still need command translation; new skills | Revisit in H2–H3 for read composition |
| **E. Single stateless Unified Digital BFF** | R1–R6 solved once; stable contracts; strangler-friendly; staff reuse | Extra hop; potential bottleneck/SPOF; logic-creep pressure | **Chosen** |

## Consequences

**Positive**
- Channel concerns are solved once for four channels, which is a direct lever on the +30% delivery target.
- Contexts stay free of UI, app-version and session concerns, and their contracts stay stable.
- Legacy can be strangled behind the BFF without app releases.
- One consistent security and error model for all channels.

**Negative / risks and mitigations**

| Risk | Mitigation |
|------|-----------|
| State or business rules creep in ("bloated controller") | Fitness tests in CI: no datastore dependency; no more than one context **command** per request; review checklist; design-authority spot checks |
| Becomes a team bottleneck | Modules per bounded context owned by that context's team (CODEOWNERS); platform team owns only the core (auth, schema serving, observability) |
| Single point of failure | Stateless, horizontally scaled, multi-AZ; per-module bulkheads and circuit breakers; schemas cached at the edge |
| Added latency | p95 latency budget per endpoint; co-location with contexts; composition in parallel |
| Pass-through endpoints add no value | **Pass-through test** (below) |

## Pass-through test (keeps the BFF honest)

A BFF endpoint must do at least one of: **secure** (R1), **negotiate/serve schemas** (R2), **translate** (R3), **compose** (R4) or **shield legacy** (R5). An endpoint that does none of these is a pass-through: route it through the internal gateway instead and delete it from the BFF. Review this quarterly.

## Review triggers

Revisit this decision if:
- more than ~30% of BFF endpoints fail the pass-through test
- the context teams' open-host services and read models make composition unnecessary
- GraphQL federation becomes an enterprise standard
- a channel develops needs that schema hints cannot express, in which case consider a thin channel adapter for that channel only
