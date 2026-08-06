# Design Case 3: The Board Wants Zero Trust, and That Is Not a Design

*Part of [The decisions](../decisions.md). Rendered version on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/case-3-zero-trust-acquisition/).*

## The scenario

The bank acquires a smaller fintech lender. The acquired company runs a flat network: once you are inside, everything is reachable. The board, briefed by vendor marketing, mandates "zero trust." Integration has to complete within 12 months.

The acquired estate includes legacy applications that cannot be modified, with no modern authentication and no mutual TLS, plus payment-adjacent systems that cannot take downtime. The team is the bank's existing security function and two engineers inherited from the acquisition.

## The decision, sentence one

**Deliver zero trust as a phased migration of trust decisions from network location to identity. Not as a product purchase, and not as a big-bang rebuild.**

The board named a framework. The architecture's job is to hand back testable properties: no implicit trust from network location, and every access authenticated, authorized, encrypted and logged. Anything that cannot be expressed as one of those four does not get budget, because it is marketing.

## Scope

**In:** access paths across both entities, covering user to application, workload to workload, and administrator to production. Unification of the workforce identity plane. Protection of the crown-jewel data stores, meaning payment flows and customer PII.

**Out:** replacing or modifying the legacy applications. The acquired firm's customer-facing product architecture. Corporate productivity networking beyond what the identity work touches. Each is a separate program with its own business case.

## Constraints

- **The legacy systems are unmodifiable.** They can be wrapped and contained. They cannot be rewritten. Any design that requires them to change is dead before it starts.
- **No downtime on payment flows.** Every migration step has to be reversible.
- **Unknown inherited posture.** The acquired estate has no reliable asset inventory, and you cannot segment what you cannot enumerate.
- **12 months.** Long enough to phase properly, short enough that the phasing order decides the outcome.
- **The board's expectations were set by vendor marketing.** Progress gets reported as risk reduction in money, never as "percent zero trust complete," a number that means nothing and can only disappoint.

## Trust boundaries

1. **User to application.** Currently defended by network location. This is the boundary the migration exists to replace with identity and device posture, evaluated per request.
2. **Workload to workload.** The flat network's real exposure. East-west movement with no internal authorization is how a single foothold becomes a breach.
3. **The acquisition boundary itself.** Two organizations, two identity systems, two sets of privileged users, one unknown asset inventory. This is the vector in every integration post-mortem worth reading.
4. **The admin plane, doubled.** Inherited privileged accounts with unknown provenance are the highest-risk access in the combined estate on day one.

## Assumptions

1. **The acquired identity provider can be federated** into the bank's. If it cannot, workforce identity unification becomes phase zero and the timeline moves. Flagged to the board, not buried in an appendix.
2. **A credible asset inventory can be built inside 60 days.** Every downstream decision inherits from this one, so it gets verified before anything else is promised.

## Design commitments

**1. Unify workforce identity before buying anything.** Federate the acquired IdP into the bank's, reconcile privileged accounts, and establish one source of truth for who anyone is. Every later control depends on identity being answerable.

*Rejected:* selecting a "zero trust platform" first. Tooling bought before identity is unified automates the confusion at scale, and does it on a three-year contract.

**2. Identity-aware ingress in front of applications.** Applications sit behind an identity-aware access proxy making per-request decisions on user identity and device posture, replacing VPN-and-network-location trust for user access. This is the pattern in my private Zero Trust reference build: identity-aware proxy at the edge, mutual TLS in the service mesh, policy-as-code authorization, all of it reproducible in Terraform.

*Rejected:* network-first microsegmentation as the opening move. It is necessary eventually. As step one it spends a year re-IP-ing a flat network, during which the flat network stays flat.

**3. Wrap the crown jewels, then migrate by data sensitivity.** Payment flows and customer PII stores get wrapped first, with proxied access, tightened egress and dedicated logging, without rebuilding them. Phases follow data sensitivity, not organizational convenience or whichever team volunteers.

*Rejected:* big-bang re-architecture. Twelve-month transformation programs that defer all risk reduction to the end tend to die at month eight with nothing shipped. Phasing that front-loads the crown jewels means a breach at month four hits a smaller target than it would have at month one, and smaller again by month seven.

**4. Legacy containment, named as what it is.** Applications that cannot do modern authentication get wrapped by the proxy where it can reach them, and contained by network-level segmentation only where it cannot. Segmentation is used as a compensating control, written down as a compensating control, and given an expiry review date.

*Rejected:* modifying the legacy applications. That is risk without ownership. The people who understood those systems left two employers ago.

## The risk statement

Acquisition integrations are the canonical breach scenario: a flat network, unknown assets and inherited privileged users, all during the exact period when everyone's attention is on the deal. A breach through the acquired entity in year one costs regulatory penalty plus impairment of the deal's rationale.

Against that, a phased identity-first program is the cheaper line item by an order of magnitude, and it buys measurable risk reduction every quarter instead of a promise at the end of the year.

## First steps

First 60 days: build the asset inventory, federate the IdP and reconcile privileged accounts, and put identity-aware ingress in front of the two crown-jewel data stores. Report to the board as exposure reduced per quarter, in money.

---

*Previous: [Case 2, approving an AI vendor for KYC](case-2-ai-vendor-kyc.md). Method and grounding: [Design Cases](../decisions.md).*
