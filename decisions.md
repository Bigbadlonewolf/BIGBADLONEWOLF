# The decisions

Four scenarios for a regional bank, worked end to end as architecture decisions rather than product selections. Each frames the problem before naming a component, and every commitment carries the alternative I rejected. An answer without a rejected alternative is a preference, not a decision.

These are design scenarios, not client engagements. The organizations are illustrative. The reference builds behind them are real, and where a build cannot do something, the case says so rather than rounding up.

## The cases

**[Design Case 1: Migrate the Payment API in 90 Days, Regulator Watching](design-cases/case-1-payment-api-migration.md)**

Tokenize at the payment service provider, so no card number ever reaches bank infrastructure. The cardholder data environment then collapses to the integration boundary, and segmentation is answered by architecture rather than by compartmentalization. *Rejected:* an in-house token vault, which hands a three-person team a regulated environment to run for 90 days.

This case also carries the decision I got wrong first. The access broker originally had its own revocation function and I deleted it, because Privileged Access Manager expires the grant on its own. What replaced it detects an overrun and does not contain one. So the defensible claim is "detected inside roughly one sweep," not "contained in fifteen minutes."

**[Design Case 2: Approving an AI Vendor for KYC Without Losing the PII](design-cases/case-2-ai-vendor-kyc.md)**

Approve with conditions, and the conditions are the design. The vendor has to read cleartext documents in order to process them, so past that boundary the control plane is contractual rather than technical. Encryption protects data in motion and at rest. It cannot protect data being processed. *Rejected:* pre-anonymizing the documents, which is broken for image processing and wrong in law for identity records the bank is required to retain.

**[Design Case 3: The Board Wants Zero Trust, and That Is Not a Design](design-cases/case-3-zero-trust-acquisition.md)**

Deliver it as a phased migration of trust decisions from network location to identity. Four testable properties, and anything that cannot be stated as one of them does not get budget. *Rejected:* buying a zero trust platform first, which automates the confusion at scale before anyone has unified identity.

**[Design Case 4: Four Thousand Hardcoded Credentials, and a Payment Pipeline That Cannot Stop](design-cases/case-4-secrets-credential-rotation.md)**

One on-premises-anchored, HSM-backed credential broker as the issuance plane for every non-human identity, with static secrets surviving only as a registered exception. The payment path fails static rather than closed: it holds a lease longer than worst-case broker recovery, so an outage blocks new issuance without stopping in-flight processing. *Rejected:* per-platform secret stores, which are cheaper up front and produce four partial audit logs where the regulator asked for one system of record.

## The bank

All four cases concern the same illustrative regional US bank, so the money figures can be compared across cases instead of each being invented locally.

- ~250,000 retail customers
- ~150,000 active debit and credit cards
- ~$2 billion in total assets
- US federal and state banking regulation (GLBA, state breach notification, OCC/FDIC oversight), plus PCI DSS for cardholder data
- Hybrid estate: on-premises core banking (mainframe and Oracle), digital channels on GCP

## The framework

Seven steps, same order every time. The order is the whole point. It forces problem-framing before component-naming, which is where architecture defenses usually fall over.

1. **Task.** What is actually being asked, and what decision is owed. For a decision problem the decision leads, in sentence one.
2. **Scope.** What is inside the design and what is explicitly outside it. Every scoped item is a promise the design has to keep.
3. **Constraints.** The givens that limit freedom: time, team, continuity obligations, political reality. Constraints are not risks. Risks are what happen despite the design. Constraints are what the design has to live inside.
4. **Trust boundaries.** Every place data, control, or trust crosses an ownership line. Three kinds: data flows, the admin plane (every privileged human is a boundary), and third parties.
5. **Assumptions.** Written down, each one load-bearing. If it turns out false, a named part of the design changes.
6. **Design commitments.** The decisions themselves, each carrying at least one rejected alternative and the reason it lost.
7. **Close.** Risk in money and consequence, then the first steps that verify the assumptions.

Two checks run before any of it counts as finished.

**Scope-to-design mapping.** Every numbered scope item gets a numbered design commitment, one to one. A requirement that appears in scope and quietly vanishes before the design section is the most common failure in an architecture defense, and it is the easiest one for an interviewer to catch.

**The mechanism test.** Every named control whose mechanism is not self-evident from the system named answers four questions: who, through what system, enforced by what, and evidenced by what artifact. A control that survives only as a noun ("encryption", "DLP", "monitoring") is a category label waiting for a decision, not the decision itself.

## Principles

What the four cases have in common, stated once so each case does not have to argue it again.

- **Frame before components.** The first product name appears only after scope, constraints, boundaries and assumptions are on the table.
- **Decisions come with rejected alternatives attached.** An answer without a rejected alternative is a preference, not a decision.
- **Match the control to the boundary.** When data has to be processed by a third party, contracts are the control plane. When it must not exist at all, architecture is.
- **The best evidence is generated, not collected.** Controls whose operation produces their own audit trail answer a regulator continuously rather than annually.
- **State risk in money.** A board that hears "seven figures of exposure against a five-figure contract" needs no security vocabulary to make the decision.
- **Reasonable assurance, never full.** Precision about what cannot be promised is what makes the promises worth signing.
- **Translate mandates into properties.** When leadership names a framework, the deliverable is testable properties, not a product shortlist.
- **State the load-bearing figures.** For any tier-0 dependency: RTO, TTL, renewal interval, each labelled measured, target, or unvalidated. If unvalidated, name the test and the date.
- **Name the compensating control.** The thing that justifies the asymmetry between normal-state and degraded-state behaviour, and when it is available.
- **Define what invalidates the decision.** Reversal triggers that supersede rather than patch, each tied to a verifiable condition and a deadline.
- **Define what proves it.** Acceptance criteria that test the degraded case, not the happy path.
- **Know when to stop.** The one-pager is the real test of clarity.
