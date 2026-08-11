# Design Case 3: The Board Wants Zero Trust, and That Is Not a Design

*Part of [The decisions](../decisions.md). Rendered version on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/case-3-zero-trust-acquisition/).*

## The scenario

The bank acquires a smaller fintech lender. The acquired company runs a flat network: once you are inside, everything is reachable. The board, briefed by vendor marketing, mandates "zero trust." Integration has to complete within 12 months.

The acquired estate includes legacy applications that cannot be modified, with no modern authentication and no mutual TLS, plus payment-adjacent systems that cannot take downtime. The team is the bank's existing security function and two engineers inherited from the acquisition.

## The decision, sentence one

**Deliver zero trust as a phased migration of trust decisions from network location to identity. Not as a product purchase, and not as a big-bang rebuild.**

The board named a framework. The architecture's job is to hand back testable properties: no implicit trust from network location, and every access authenticated, authorized, encrypted and logged. Anything that cannot be expressed as one of those four does not get budget, because it is marketing.

```mermaid
flowchart LR
    subgraph BEFORE["Before - trust from network location"]
        F["One foothold"]
        F --> A1["App A"]
        F --> A2["App B"]
        F --> DB["Database"]
        F --> PAY["Payments"]
        F --> ADM["Admin"]
        F --> PII["PII store"]
    end

    subgraph AFTER["After - identity, evaluated per request"]
        U["User and device"] --> IAP["Identity-aware proxy<br>decides on every request"]
        IAP --> W1["Workload"]
        W1 -- "mTLS" --> W2["Workload"]
        IAP --> LEG["Legacy app<br>wrapped, or segmented"]
    end

    PROPS["The four testable properties - anything else does not get budget<br>1 No implicit trust from network location - 2 Every access authenticated<br>3 Every access authorised and encrypted - 4 Every access logged"]

    classDef key fill:#F7C948,stroke:#EAB938,color:#1A1A1A
    class F,IAP key
```

*The left half is not a diagram of a bad network. It is a diagram of a network
where **being inside is the authorisation**, which was reasonable when the
perimeter was the building. The right half moves that decision to identity and
makes it per request. The box underneath is what the board's word has to become
before anyone can be held to it.*

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

**Mechanism test**, for the two words doing the most work — "tightened egress" and "dedicated logging."

- *Who:* a platform engineer maintains the allowed endpoint list.
- *Through what:* VPC Service Controls, Cloud Armor and VPC firewall rules.
- *Enforced by:* organisation policy preventing log deletion, and firewall rules denying all egress except to enumerated payment endpoints on 443.
- *Evidenced by:* Cloud Audit Logs and VPC Flow Logs streamed to an immutable SIEM bucket on seven-year WORM retention, with a weekly review of denied-egress attempts and a quarterly access review of the endpoint list.

*Rejected:* big-bang re-architecture. Twelve-month transformation programs that defer all risk reduction to the end tend to die at month eight with nothing shipped. Phasing that front-loads the crown jewels means a breach at month four hits a smaller target than it would have at month one, and smaller again by month seven.

```mermaid
flowchart LR
    P0["PHASE 0 - days 0 to 60<br>Asset inventory<br>IdP federation<br>Privileged accounts reconciled<br><br>Blast radius ████████"]
    P1["PHASE 1<br>Crown jewels wrapped<br>Payment flows, PII stores<br>Proxied, egress tightened<br><br>Blast radius ██████"]
    P2["PHASE 2<br>Identity-aware ingress<br>broadened in order of<br>data sensitivity<br><br>Blast radius ████"]
    P3["PHASE 3<br>Legacy containment<br>Compensating control,<br>with an expiry review<br><br>Blast radius ██"]

    P0 -- "Quarter 1" --> P1
    P1 -- "Quarter 2" --> P2
    P2 -- "Quarter 3" --> P3

    classDef key fill:#F7C948,stroke:#EAB938,color:#1A1A1A
    class P0 key
```

*The phases are ordinary. **The falling blast radius is the argument.** A
programme that delivers its risk reduction in the last quarter is a programme
that carries full exposure through three quarters of an integration, which is
exactly the window in which acquisitions get breached. Reported to the board as
exposure reduced in money, never as percent complete.*

**4. Legacy containment, named as what it is.** Applications that cannot do modern authentication get wrapped by the proxy where it can reach them, and contained by network-level segmentation only where it cannot. Segmentation is used as a compensating control, written down as a compensating control, and given an expiry review date.

**Mechanism test.**

- *Who:* a network engineer configures the containment.
- *Through what:* an isolated legacy VPC with no route to the crown-jewel VPC except through an explicit proxy on 443 to known endpoints.
- *Enforced by:* VPC firewall rules and Cloud Armor, with a quarterly penetration test that has to demonstrate no lateral movement from the legacy VPC into the crown-jewel VPC.
- *Evidenced by:* the pen-test report and the firewall rule audit log, plus a 90-day expiry review asking whether the proxy can reach the application yet.

*Rejected:* modifying the legacy applications. That is risk without ownership. The people who understood those systems left two employers ago.

**5. Workload-to-workload identity, and an honest fallback where identity will not fit.** Where workloads can carry identity — containerised, in a service mesh — it is mTLS with SPIFFE/SPIRE identities and authorization policy enforced by the mesh. Where they cannot — legacy VMs, bare metal — it is host-based firewall rules with an explicit allow-list, every denied flow logged to the SIEM and reviewed quarterly.

**Mechanism test.**

- *Who:* a platform engineer defines service identity and policy.
- *Through what:* the service mesh for modern workloads, host-level enforcement for the rest.
- *Enforced by:* deny-all with explicit allow, in mesh policy or host firewall rules, with denied flows raising a SIEM alarm.
- *Evidenced by:* the policy repository holding version-controlled allow-lists, the denied-flow log, and a quarterly review of attempted unauthorized east-west connections.

*Rejected:* assuming east-west security is handled by "being in the same VPC." That is the flat-network assumption restated in cloud vocabulary, and it is the assumption that created this engagement.

**6. One admin plane for the combined estate.** Inherited privileged accounts are reconciled in week one, and every elevation — inherited or original — runs through the same tracked, time-boxed, logged path as [Case 1](case-1-payment-api-migration.md): a request, a recorded approval, an IAM Condition carrying an expiry, and an append-only ledger. The same limit applies here as there. The reconciliation sweep detects a grant that outlives its window. It does not revoke it.

*Rejected:* running two admin-plane systems, one per legacy organisation. That preserves exactly the doubled blast radius the acquisition created, and it is the default outcome if nobody makes this decision explicitly.

**7. Every migration step reversible.** Each cutover reverses through Terraform state rollback and a DNS cutback. Payment flow cutover is blue-green with a 60-second DNS TTL.

**Mechanism test.**

- *Who:* a platform engineer runs the rollback runbook.
- *Through what:* Terraform state revert plus a DNS record update.
- *Enforced by:* automated canary analysis that blocks full cutover when the error rate exceeds 0.1%.
- *Evidenced by:* a quarterly rollback drill report, with a recovery time under five minutes for the payment flow revert.

**8. Board reporting as a generated artifact, not a slide someone writes.** A quarterly exposure report: unprotected crown-jewel access paths, denied-flow incidents, privileged access grants, each with a trend and a monetary exposure figure using the Case 1 derivation method. Generated on a schedule from SIEM and IAM audit data, so nobody is tempted to round it.

## The risk statement

Acquisition integrations are the canonical breach scenario: a flat network, unknown assets and inherited privileged users, all during the exact period when everyone's attention is on the deal.

**Derivation,** on the same method as Case 1. State breach notification at $5–$20 per record across 250,000 customers ($1.25M–$5M), plus OCC or FDIC enforcement ($1M–$10M),[^1] plus impairment of the deal's rationale as a goodwill write-down. **Seven figures to mid-eight figures.**

Against that, a phased identity-first program is the cheaper line item by an order of magnitude, and it buys measurable risk reduction every quarter instead of a promise at the end of the year.

## Reversal triggers

This design is superseded, not patched, if any of these is true by day 60.

1. **The acquired IdP cannot be federated.** Workforce identity unification becomes phase zero, the 12-month timeline moves, and the board hears it at the time rather than at month ten.
2. **The 60-day asset inventory fails.** Every downstream decision inherits from it, so without it the segmentation promises are unfulfillable and should not be made.
3. **A crown-jewel store turns up that can be neither wrapped nor contained.** A payment-adjacent system with no proxy path and no segmentation option needs its own exception record and a QSA review, not a footnote in this one.

## Acceptance criteria

1. **Federation works end to end.** An acquired engineer signs in to a bank application through federated identity with a device posture check, inside 60 days. Evidence: the identity provider audit log.
2. **East-west denial is real.** A simulated compromised workload tries to reach the crown-jewel store and is denied, with the denied-flow log showing source workload identity and timestamp. Evidence: the SIEM query and the pen-test report.
3. **One admin plane, both estates.** An inherited privileged account elevates through the same broker as an original account, with the reconciliation timestamp recorded and no standing grants left behind. Evidence: the append-only ledger.
4. **Rollback drill.** Payment flow cuts over to green, then reverts to blue in under five minutes with zero failed transactions. Evidence: the drill report.
5. **The board report generates itself.** The first quarterly exposure report is produced on schedule from SIEM and IAM logs, showing unprotected path count and the monetary exposure trend. Evidence: the published report and its data lineage.

## First steps

First 60 days: build the asset inventory, federate the IdP and reconcile privileged accounts, put identity-aware ingress in front of the two crown-jewel data stores, and publish the first quarterly board report as a baseline. Report to the board as exposure reduced per quarter, in money.

[^1]: OCC, *In the Matter of Capital One, N.A.*, August 2020, an $80 million civil money penalty for deficiencies in cybersecurity and internal controls. The $1M–$10M range used here for a bank of roughly $2B in assets is extrapolated from larger enforcement actions rather than drawn from a comparable-size precedent, and the actual figure in any specific case turns on severity, duration and wilfulness.

---

*Previous: [Case 2, approving an AI vendor for KYC](case-2-ai-vendor-kyc.md). Next: [Case 4, secrets management and credential rotation](case-4-secrets-credential-rotation.md). Method and grounding: [Design Cases](../decisions.md).*
