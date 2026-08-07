# Design Case 1: Migrate the Payment API in 90 Days, Regulator Watching

*Part of [The decisions](../decisions.md). Rendered version on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/case-1-payment-api-migration/).*

## The scenario

A regional bank runs its customer-facing payment API on-premises. Leadership wants the workload on GCP within 90 days. The regulator requires evidence that cardholder data is segmented, and that privileged access to production is time-bound and auditable. The migration team is three people.

## Scope

**In:** the payment API tier and its data path, the cardholder data environment boundary, the privileged access model for production, and evidence generation for both regulatory requirements.

**Out:** the core banking system, which stays on-premises. Database replatforming. Every other workload. A 90-day window that attempts more than this delivers nothing.

## Constraints

- **90 days.** Sequential strategies fail by construction. Design and evidence collection run in parallel from week one.
- **Team of three.** Every design choice gets taxed for operational cost. At this staffing level, managed services beat self-managed almost every time.
- **Migration, not greenfield.** The API keeps serving customers while it moves, and it keeps a dependency back to core banking on-premises.
- **The regulator is a primary stakeholder.** Evidence is a design output, not something produced in a panic the week before an audit.

## Trust boundaries

1. **Customer device to internet.** Nothing on the far side is controlled. Clients are assumed hostile.
2. **Internet to GCP edge.** Inspection, rate limiting and DDoS absorption live here, at the provider edge and Cloud Armor, not at the API gateway.
3. **Edge to service tier.** Authenticated, authorized, logged.
4. **Service tier to cardholder data environment.** The compliance boundary. The design goal is to make this boundary defend itself by shrinking what sits behind it.
5. **The admin plane.** Every human who can touch production is a trust boundary with a name on it. This one gets its own design, not a bullet point.

## Assumptions

1. **The bank may not need to store PAN at all.** If a payment service provider can tokenize at capture, the cardholder data environment shrinks toward zero. This is the highest-value question in the engagement and it gets verified in week one.
2. **A GCP organization with at least a minimal landing zone exists.** If it does not, standing one up is week-one work rather than an afterthought. Every control below inherits its posture from that foundation.

## Design commitments

**1. Single managed ingress.** A managed API gateway fronts the API, with Cloud Armor at the provider edge for DDoS and Layer 7 filtering.

*Rejected:* a self-managed gateway (Kong, Envoy on GCE or GKE). A three-person team cannot safely operate ingress infrastructure and execute a migration at the same time. The risk is not the build. It is the operate.

**2. Tokenization at the payment service provider, before PAN reaches bank infrastructure.** Payment fields are served directly from the PSP through hosted fields or a client SDK. The raw card number travels from the customer's device over TLS straight into the PSP's vault, the vault returns a token, and the client submits the token. Every bank-side system, gateway, service tier, database and log sees tokens only. De-tokenization for settlement happens inside the PSP's vault, on the PSP's compliance scope.

That answers the regulator's first question by construction. With no PAN in the environment, the cardholder data environment collapses to the integration boundary, and the segmentation obligation is met by architecture rather than by compartmentalization alone.

DLP then moves to its correct job. It is not there to block exfiltration. It is there to prove the negative: continuous scanning for card-number patterns turns "no PAN exists in our systems" from an assertion into a monitored, logged, testable claim.

*Rejected:* running an in-house token vault. Full control and no provider dependency, at the cost of owning a regulated CDE with its own segmentation, monitoring and assessment scope. A three-person team cannot carry that in 90 days.

```mermaid
flowchart LR
    D["Customer device<br>assumed hostile"]

    subgraph PSP["PSP compliance scope - the CDE"]
        HF["PSP hosted fields<br>served by the PSP"]
        V["PSP token vault<br>de-tokenises for settlement"]
    end

    subgraph GCP["Bank on GCP - tokens only"]
        CA["Cloud Armor<br>edge, L7 filtering"]
        GW["Managed API gateway<br>the single ingress"]
        SVC["Service tier<br>tokens only"]
        ST["Store and logs<br>DLP scans these to prove the negative"]
    end

    CORE["Core banking<br>on-premises, out of scope"]

    D -- "PAN over TLS" --> HF
    HF --> V
    V -. "token returned to the client" .-> D
    D -- "token" --> CA
    CA --> GW
    GW --> SVC
    SVC --> ST
    SVC -. "retained dependency" .-> CORE

    linkStyle 0,1 stroke:#EAB938,stroke-width:3px
```

*The whole argument is the boundary. **The card number never enters the bank's
half of this diagram**, so the cardholder data environment sits inside the
provider's compliance scope and the segmentation obligation is met by the shape
of the design rather than by compartmentalising a problem it chose to keep.*

**3. Zero standing privileged access, with elevation that expires on its own.** Engineers hold read-only roles by default. Elevation runs through a tracked workflow: a request, a recorded approval from a second person, a time-boxed grant scoped by IAM Condition to the one resource in question, and every request, approval, grant and expiry written to an append-only ledger. Break-glass exists for real emergencies, needs two people, alarms on use, and gets reviewed the next morning.

One detail here is worth more than the rest of the design, because it is the part I got wrong first.

The reference build originally carried its own revocation function. I deleted it. Google's Privileged Access Manager expires the grant on its own, which left a second revocation path whose only remaining justification was that it already existed. What replaced it is a read-only reconciliation job that sweeps for grants outliving their window and writes a flag to the ledger.

That job detects an overrun. It does not contain one. Nothing is revoked automatically, and that is a decision rather than a gap. Automated revocation is an action taken against a production access-control plane, and an action needs an alerting and rollback story before it runs unattended. So the claim I can defend to a regulator is "detected inside roughly one sweep," not "contained in fifteen minutes." Two different sentences, and only one of them is true.

What survives is the useful part: the audit log of the grant lifecycle *is* the regulator's evidence. The control and the proof are the same artifact, which is the strongest available answer to "time-bound and auditable."

```mermaid
flowchart LR
    E["Engineer<br>read-only by default"] --> R["Request<br>with justification"]
    R --> A["Approval<br>a second person"]
    A --> G["PAM grant<br>IAM Condition, one resource"]
    G --> X["Expiry<br>PAM, on its own"]

    E -. "emergencies only" .-> BG["Break-glass<br>two people, alarms on use<br>reviewed the next morning"]

    G -. "reads grant state" .-> S["Reconcile sweep<br>every 15 minutes<br>read-only on PAM"]
    S --> DET["Detects an overrun.<br>Does not contain one."]
    DET -- "EXPIRE_FLAG row" --> L

    DEL["revoke_access - DELETED<br>PAM already owns expiry, and two<br>enforcement paths can disagree"]

    R -. "every event, written once" .-> L["Append-only ledger<br>request, approval, grant, expiry, every flag<br>the control and the evidence are the same artifact"]
    A -.-> L
    G -.-> L
    X -.-> L

    classDef key fill:#F7C948,stroke:#EAB938,color:#1A1A1A
    classDef gone fill:none,stroke:#9A9A9A,stroke-dasharray:5 4,color:#6E6E6E
    class G,DET key
    class DEL gone
```

*The deleted box is the part worth defending in an interview. Two enforcement
paths that can disagree is how you get an incident where each one assumes the
other handled it, so the second path went rather than being kept for having
already been written. What replaced it **detects and does not contain**, which
is a narrower claim and a true one.*

*Rejected:* bastion hosts with standing SSH access. Unauditable standing privilege, which fails the second regulatory requirement by design.

**4. Continuous compliance evidence.** Infrastructure changes pass policy-as-code evaluation in CI before merge, the pattern built in [Compliance as Code](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE): PCI DSS v4.0, SOC 2 and NIST 800-53 mappings, gated jobs, blocking on violation. Compliance posture becomes continuous and version-controlled instead of annual and screenshot-based.

The limit belongs in the design document rather than in a footnote discovered later. A policy engine reading Terraform plan output can enforce "you must declare a value." It cannot enforce "that value is truthful," because verifying that needs runtime knowledge the plan JSON does not contain. Anything in that gap needs a different control, not a weaker version of this one.

## The risk statement

A failed segmentation assessment means failed audits, card-brand penalties, and a breach blast radius where one compromised credential becomes a reportable event. Realistically a seven-figure exposure, against controls whose incremental cost is a fraction of that.

The two decisions compound. Even a fully compromised admin credential lands in an environment holding no card numbers. Each control shrinks the other's worst case, which is the argument for doing both rather than picking the cheaper one.

## First steps

Week one: verify the PAN-storage assumption, stand up or validate the landing zone, put the on-premises connectivity question on the table, and open the PSP contract review.

---

*Next: [Case 2, approving an AI vendor for KYC](case-2-ai-vendor-kyc.md). Method and grounding: [Design Cases](../decisions.md).*
