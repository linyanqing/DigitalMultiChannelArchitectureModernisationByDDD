# 04 – As-Is User Experience

## Personas

| Persona | Description | Goals | Frustrations |
|---------|-------------|-------|-------------|
| **Mia – New café owner (sole trader)** | Opening a café, time-poor, mobile-first | Take card payments ASAP, simple setup | Printing/scanning forms, not knowing status, repeated requests for documents |
| **Raj – Multi-site retailer (company, 3 directors)** | Existing merchant, uses web portal on desktop and app on the go | Keep details current, add terminals, change bank account quickly | Some changes only on paper, app lacks features that web has, calling support to check progress |
| **Priya – Onboarding Ops specialist (staff)** | Processes applications and changes | Clean, complete data; fewer chases | Re-keying, multiple staff portals, no single case view |
| **Tom – POS/Partner BDM** | Refers merchants | Fast approvals for referred merchants | No visibility of application status |

---

## Journey map – New merchant onboarding (Mia)

| Stage | Discover | Apply | Submit | Wait / Verify | Approve & set up | First transaction |
|-------|----------|-------|--------|---------------|------------------|-------------------|
| **Doing** | Finds the provider online / via partner | Downloads PDF, fills in by hand | Signs, scans, emails | Waits; receives calls/emails for missing items | Gets approval email | Terminal arrives, activates |
| **Channel** | Web (marketing) | Paper | Email / post | Phone / email | Email | Terminal, then app/web |
| **Thinking** | "Looks good, how do I start?" | "Why can't I do this on my phone?" | "Did they get it?" | "What's happening? What else do they need?" | "Finally" | "Now I need to download an app and log in separately" |
| **Feeling** | 🙂 | 😕 | 😟 | 😠 | 😐 | 🙂 |
| **Pain points** | No digital start | Long form, no pre-fill (ABN), no guidance | No confirmation / tracking | NIGO loops, days lost | No clear timeline | Disconnected from application |
| **Opportunities** | "Apply now" on web & app | Guided digital form, ABN/address lookup, save & resume | Instant confirmation, e-sign | Real-time status, automated IDV, doc upload on phone | STP for simple cases | Seamless hand-off to logged-in app |

## Journey map – Change settlement bank account (Raj)

| Stage | Need | Find how | Do it | Verify | Confirm |
|-------|------|----------|-------|--------|---------|
| **Doing** | Changing bank | Looks in app → not there; checks web → link to PDF | Prints form, all directors sign, attaches bank statement | Support calls back to verify | Email days later |
| **Thinking** | "Should be a 2-min task" | "Why is it in web but not the app?" | "Three signatures for this?" | "Is this call legit?" | "Did it take effect before settlement?" |
| **Feeling** | 🙂 | 😕 | 😠 | 😟 | 😐 |
| **Opportunities** | – | Same journey in app & web | Digital form, pre-filled, multi-party e-sign | In-app step-up auth (biometric/OTP) + automated account verification | Push/email notification, status in app |

## Journey map – Ops specialist (Priya)

| Stage | Receive | Triage | Re-key | Verify | Resolve |
|-------|---------|--------|--------|--------|---------|
| Pain | Shared inbox, attachments | Manual completeness check | Multiple domain staff portals, different UIs | Manual lookups (ABN, ID) | Manual emails; no audit trail linking steps |
| Opportunity | Single work queue with structured cases | Auto-validation at capture | Data flows straight to domain APIs | Automated IDV/KYB results attached | Templated notifications, full audit |

---

## Cross-channel experience inconsistencies (heuristic audit – validate)

| Capability | Web (JSP) | iOS (Swift) | Android (Kotlin) |
|-----------|-----------|-------------|------------------|
| Start onboarding | PDF link | Not available | Not available |
| Update contact details | ✅ (own rules) | ✅ (different validation) | ❌ |
| Change settlement account | PDF | ❌ | ❌ |
| Add director/signatory | PDF | ❌ | ❌ |
| Request extra terminal | Web form → email | ✅ (iOS-specific API) | ❌ |
| Status tracking | ❌ | ❌ | ❌ |
| Look & feel | Legacy, not responsive | Native, own design | Native, own design |
| Accessibility (WCAG 2.2 AA) | Partial | Partial | Partial |
| Authentication | Password + OTP | Biometric | Biometric |

## Language observed (feeds the glossary)

Merchants and staff say *application*, *director*, *signatory*, *settlement account* and *change request*. Channels and systems say *customer*, *profile*, *bank account*, *case* and *form*. The target uses the business terms defined in the [ubiquitous language](11-ubiquitous-language.md) consistently in UI copy, schemas and APIs.

## UX principles for the target
1. **Start anywhere, finish anywhere** – save & resume across channels.
2. **Ask once** – pre-fill from data we already hold or can look up (ABN, address, bank).
3. **Show progress** – status and next steps visible in all channels + notifications.
4. **Consistent by default, different by design** – same journey rules, data and content; channel-native interaction where it adds value (camera ID capture, biometrics, push).
5. **Accessible and inclusive** – WCAG 2.2 AA across channels via shared design system.
6. **Assisted when needed** – seamless hand-off to staff with full context.
