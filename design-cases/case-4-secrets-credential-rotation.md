# Design Case 4: Four Thousand Hardcoded Credentials, and a Payment Pipeline That Cannot Stop

*Part of [The decisions](../decisions.md). Rendered version on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/case-4-secrets-credential-rotation/).*

## Executive summary

**Decision.** Adopt a single on-premises-anchored, HSM-backed credential broker as the default issuance plane for every non-human identity. Static secrets survive only as a documented exception, each with an owner, a review date and an elimination plan.

**Four load-bearing figures.**

- Broker recovery time objective, **4 hours** — *target, unvalidated*. Primary datacentre loss to promoted replica serving issuance, including manual HSM unseal and confirmation of promotion.
- Payment-path credential TTL, **8 hours** — 4 hours of RTO plus 4 hours of margin for detecting a renewal failure and getting a human to it.
- Payment-path TTL ceiling, **24 hours**. Above this, fix the RTO or abandon the architecture.
- GCP estate default TTL, **1 hour** — longer than the on-premises default of 15 minutes, so a partitioned interconnect does not stop work.

**Compensating control.** Immediate revocation after rotation while the broker is healthy. During an outage, an out-of-band DBA runbook (dual-authorised, alarmed) and a sealed break-glass credential under two-of-three quorum provide kill capability without transiting the broker.

**Cost.** Roughly $150K–$250K a year in licensing plus 1.5 FTE of operations, against a seven-figure exposure that today is neither measured nor controlled.

**Ninety-day proof.** One payment-adjacent service in production on brokered credentials, plus a rotate-and-revoke drill on Oracle, a PassTicket or dual-userid drill on the mainframe, a broker-outage survival test, a DR promotion drill, and a load test that includes a thundering herd.

## The scenario

The bank runs a hybrid estate. Core banking and card processing stay on-premises on a mainframe and Oracle. Customer-facing digital channels, analytics and ML workloads run on GCP.

The CISO has found more than 4,000 hardcoded credentials: database connection strings in config files, API keys in GitHub repositories, service account keys embedded in microservices, and mainframe passwords rotated by hand every 90 days by a single admin who has just given notice.

The board wants a unified secrets architecture with automatic rotation for all non-human identities, zero hardcoded secrets in source or config within six months, no-downtime rotation on a payment pipeline carrying a 99.99% uptime SLA, a full audit trail for regulator review, and hybrid coverage treated as one trust domain rather than two.

The team is two platform engineers, one mainframe operator who is sceptical of cloud, and a DevOps lead who thinks Kubernetes secrets are good enough. Ninety days to deliver the architecture and prove it works.

## The decision, sentence one

**Adopt a single credential broker, anchored on-premises and HSM-backed, as the default issuance plane for all non-human identities, with static secrets as a documented exception.** The broker issues short-lived credentials against workload identity. Payment workloads hold leased credentials with a TTL longer than worst-case broker recovery, so broker unavailability blocks *new issuance* without stopping *in-flight processing*.

## Scope

**In:** non-human identities only — service accounts, database credentials, API keys, machine certificates. Both estates. The broker control plane. The audit pipeline. One proven end-to-end pattern per credential class: GCP-native, Kubernetes, Oracle, mainframe, third-party API key.

**Out:** human privileged access, which is PAM — a different product, a different project and a different budget, stated on day one so it does not creep in by week six. End-user and customer authentication. Encryption key management: cloud KMS and the on-premises HSMs stay where they are, and the broker consumes them rather than redesigning them. And migrating the other ~3,900 credentials, which is the six-month programme. The 90 days buys the architecture and five proven patterns. Migration after that is repetition, and repetition is a platform-engineering backlog, not architecture.

## Constraints

- **90 days** for the architecture plus five proven patterns. Full-estate migration is a subsequent programme, said out loud at the start.
- **99.99% SLA**, meaning 52.6 minutes of downtime a year. Rotation on the payment pipeline has to be genuinely online: no maintenance window, no rolling restart that drops in-flight transactions.
- **Two platform engineers**, which is more like one and a half after operational load, against five patterns, 90 days and a knowledge-transfer emergency.
- **No-downtime rotation requires the consumer to tolerate two valid credentials at once.** This is the constraint everyone underestimates. Rotation is not hard. Rotation without a window is hard, and the difficulty lives in the application, not in the vault.
- **PCI scope.** Putting the broker inside the CDE expands audit scope. Putting it outside means the CDE trusts an external issuer. Pick deliberately, document it, and get the QSA's view before building rather than after.
- **The mainframe operator departs on day 20.** Not a 90-day item — a fixed date. PassTicket feasibility or the dual-userid fallback has to be proven before it.
- **Political reality.** The DevOps lead thinks Kubernetes secrets are good enough and the mainframe operator distrusts cloud. Both have to be converted, not overruled. Overruled people implement the letter of a design and none of its intent.

## Trust boundaries

1. **Human and non-human, hard-separated.** No human has a path to read a machine credential in cleartext. Not the architect, not break-glass. Break-glass issues a *new* short-lived credential with a loud alarm; it never reveals an existing one.
2. **On-premises and GCP.** "One trust domain" means one identity plane, not a flat network. On-premises workloads reach GCP through workload identity federation, exchanging an on-premises assertion for a short-lived GCP token. No exported service account keys, ever, enforced by an organisation policy constraint that is itself auditable in one place.
3. **Broker control plane and data plane.** The broker's own root credentials and unseal material sit behind the HSM boundary, and are the last thing in the estate still under human quorum control.
4. **CI/CD, treated as hostile.** There are already API keys in GitHub repositories, which means the build system is a proven exfiltration path. Pipelines authenticate as workloads over OIDC and receive scoped credentials measured in minutes. Long-lived secrets in CI variables are banned, not discouraged.
5. **The RACF cryptographic boundary.** The secured signon key never leaves the mainframe, and GCP never holds mainframe credential material. This is also the sentence that brings the mainframe operator across: the answer is "we stop storing your password anywhere," not "your password now lives in the cloud."
6. **The CDE boundary around the payment pipeline.** Where the broker sits relative to it is documented and QSA-reviewed before anything is built.

## Assumptions

1. **Payment pipeline components are modifiable** — source available, and a release path that does not take a quarter. If the payment application is a vendor black box reading a password from a properties file, the design changes materially: it becomes file injection with atomic swap and a process signal, and "no downtime" then depends on vendor behaviour nobody here controls.
2. **Oracle is patched to a level supporting gradual password rollover**, and the connection pool re-authenticates inside the rollover window without a restart. Verified by day 14.
3. **The mainframe supports PassTickets** and the operator engages. The secured signon key never leaves the LPAR. Verified by day 7; if false, the dual-RACF-userid fallback has to be proven before the operator leaves.
4. **Dedicated Interconnect, not VPN**, exists on-premises to GCP with headroom, and an internal PKI and HSM are already in production.
5. **The regulator will accept the broker's issuance log as the system of record** for credential lifecycle. Ask early. If they want per-platform native logs as well, the logging design doubles.
6. **Someone owns budget for a commercial broker.** If the answer is "use open source with no support contract," the tier-0 dependency decision gets revisited, because an unsupported tier-0 dependency on the payment path is worse than the current state.

## Design commitments

**1. Elimination before custody.** Every credential in the inventory is classified into one of three buckets: eliminate (replaced by brokered or federated identity), broker dynamically (issued per session, TTL in minutes), or custody and rotate (static, vaulted, automatically rotated, exception-registered with an owner and a review date). The target is that the third bucket is small and every entry in it is named. Its size is the headline metric, because it measures the thing that matters more honestly than "secrets removed from git" does.

*Rejected:* migrating all 4,000 credentials into a vault unchanged. That builds an expensive password manager and concentrates the blast radius instead of reducing it. Most of those credentials should not exist in the target state at all.

**2. One control plane, anchored on-premises.** A broker running in the bank's datacentre, authenticating Kubernetes, GCP, certificate and mainframe-adjacent workloads, with dynamic secret engines for the databases. GCP Secret Manager stays in play only where a native integration is genuinely cheaper, and even then the broker remains the system of record. The architectural decision — one broker versus several stores — stands independently of the product; the reference implementation here is Vault Enterprise, with an alternatives evaluation if licensing is not approved by day 21.

**Mechanism test.**

- *Who:* a platform engineer configures the broker cluster and its secret engines.
- *Through what:* an on-premises primary with HSM auto-unseal, a regional read replica for latency, and a DR replica that can be promoted when the primary is lost.
- *Enforced by:* the organisation policy `iam.disableServiceAccountKeyCreation` blocking exported GCP keys, and broker policies denying read on the payment path to every identity except the payment workload's.
- *Evidenced by:* the broker audit log covering every authentication, every secret engine mount and every policy change, plus the HSM audit log covering unseal events.

*Rejected:* per-platform secret stores — Secret Manager on GCP, something else on-premises, native rotation where it exists. Cheaper up front, and it produces four partial audit logs and a reconciliation exercise. It fails the regulator's single-system-of-record requirement outright, and it fails on Oracle dynamic credentials and mainframe integration without custom bridging that nobody will own.

*Rejected:* Kubernetes Secrets as the default. Base64 in etcd, no read audit, no expiry, no rotation. That is storage, not a control.

```mermaid
flowchart LR
    subgraph ONPREM["On-premises - the anchored control plane"]
        HSM["HSM boundary<br>root and unseal keys<br>human quorum, two of three"]
        BRK["Credential broker<br>primary, HSM auto-unseal<br>dynamic secret engines"]
        RR["Read replica<br>latency"]
        DR["DR replica<br>4h RTO, target"]
    end

    subgraph GCPZ["GCP"]
        WIF["Workload identity federation<br>short-lived token<br>no exported service account keys"]
    end

    MF["Mainframe - outside the plane<br>RACF PassTickets<br>secured signon key never leaves the LPAR"]

    HSM --> BRK
    BRK --> RR
    BRK --> DR
    BRK --> WIF
    BRK -. "holds no mainframe credential material" .-> MF
    BRK --> ORA["Oracle<br>dynamic credentials"]
    BRK --> K8S["Kubernetes<br>sidecar, memory mount, not etcd"]
    BRK --> KEY["Third-party keys<br>custody plus rotation"]
    BRK --> AUD["Immutable SIEM, 7-year WORM<br>lease ID and workload identity only"]

    linkStyle 0 stroke:#EAB938,stroke-width:3px
```

*One issuance plane, five consumption patterns. The line that matters is the
first one. **The broker's own root material is the last thing in the estate
still under human quorum**, and everything downstream of it is a credential
that expires on its own. The mainframe sits outside the plane on purpose,
because the secured signon key never leaves the LPAR.*

**3. Payments fail static, not closed.** This is the most important decision in the case, so it gets stated in full. The payment pipeline holds a leased credential whose TTL exceeds worst-case broker recovery time, renews in the background, and keeps processing on a valid unexpired lease when the broker is unreachable. It stops only when the credential itself expires.

The figures, each labelled:

| Figure | Value | Basis |
| --- | --- | --- |
| Broker RTO | 4 hours | **Target, unvalidated.** Primary loss to promoted replica, including manual HSM unseal |
| On-premises default TTL | 15 minutes | Estate default |
| GCP default TTL | 1 hour | Survives an interconnect partition without renewal |
| Payment-path TTL | 8 hours | 4h RTO plus 4h margin for detection and human response |
| Payment-path renewal | Every 30 minutes | Shortens time-to-detect, not the exposure window |
| Payment-path ceiling | 24 hours | Above this, fix the RTO or supersede the design |
| Revocation window | Immediate when healthy, up to 8 hours when not | The asymmetry this design is accountable for |

A 30-minute renewal interval does not shorten the life of a healthy credential — the payment path permanently holds seven and a half to eight hours of remaining validity. What it buys is early detection of a broker problem.

The case for the asymmetry is what payments has *today*: a hardcoded password, shared, unrotated, unattributed in any audit log, unrevocable without a config push, sitting in a file on a server. An 8-hour credential that is unique per workload, tied to a lease ID, attributable in the audit log and revocable is not a concession. It is a step change, and pretending otherwise would be arguing against the design from a baseline that does not exist.

*Rejected:* a shorter payment-path TTL, which would require automated replica promotion with no human confirmation. That introduces split-brain risk on the credential plane for payments — trading a theoretical reduction in exposure for a real availability and correctness risk on the money path.

```mermaid
flowchart LR
    T0["t = 0<br>last successful renewal"] --> O["Broker outage begins<br>4h RTO - target, unvalidated"]
    O --> S["Hours 0 to 4<br>payments keep processing<br>new issuance blocked"]
    S --> M["Hours 4 to 8<br>margin: detect the failure,<br>get a human to it"]
    M --> E["Hour 8<br>lease expires, payments stop"]

    O -. "revocation cannot transit the broker" .-> K["Out-of-band kill paths<br>dual-authorised DBA runbook<br>sealed break-glass, two of three quorum"]
    E -. "if a measured RTO needs longer" .-> C["24h ceiling<br>fix the RTO instead"]

    linkStyle 1,2 stroke:#EAB938,stroke-width:3px
```

*The highlighted span is the whole argument. **The lease outlives the outage**,
with four hours left over for someone to notice and act. The ceiling on the
right is the honest limit: if a measured recovery time ever pushed the required
lease past it, the answer is to fix the recovery time rather than widen the
lease.*

**4. No-downtime rotation, proven by revocation rather than by a green pipeline.** The acceptance test is to rotate the payment pipeline's database credential in production with transactions in flight, revoke the old credential immediately, and show zero failed transactions with a complete audit trail. A rotation nobody attacked is not evidence of anything.

- **Oracle.** Gradual password rollover keeps two credentials valid through the transition, which is what lets a legacy connection pool survive it.
- **Mainframe.** PassTickets: single-use, time-bounded credentials derived from a secured signon key that never leaves the LPAR, so no reusable password is stored anywhere. Fallback is dual RACF userids — rotate the idle one, verify with a test transaction, flip the proxy target, and flip back if it goes wrong.
- **GCP service accounts and API keys.** The broker's native engine rotates them, with two valid keys existing briefly during cutover.

*Rejected:* proxy-level credential swap against a *single* mainframe userid as the PassTicket fallback. A failed rotation there revokes the production payment userid with no way back, which is a fallback that can cause the outage it exists to prevent.

**5. Out-of-band kill paths, because the 8-hour TTL creates a window.** During a broker outage that credential cannot be revoked through the broker, so two paths exist that do not transit it.

- **An Oracle lock-or-repassword runbook**, documented and tested, executed by two authorised DBAs and alarmed in the SIEM. This is a deliberate, documented exception to the hard human/non-human separation in trust boundary 1: a standing privileged human route to a machine credential, governed by dual authorisation, a standing alarm and quarterly attestation of who holds it. Writing it down as an exception is the difference between a compensating control and a hole.
- **A sealed break-glass credential** for the Oracle payment account, held offline under two-of-three quorum. Use triggers incident response and regulator notification within four hours.

**Mechanism test**, for break-glass.

- *Who:* two of three named key holders.
- *Through what:* a physical safe or HSM-encrypted offline storage.
- *Enforced by:* dual authorisation to unseal, a SIEM alarm on any use, and regulator notification inside four hours.
- *Evidenced by:* the usage log (timestamp, key holders, incident ticket) and a quarterly attestation signed by the CISO naming the current holders.

**Recovering broker authority afterwards.** Once DBAs have repassworded the Oracle account out of band, the broker's stored credential is stale, and on recovery it will try to issue against a password that no longer exists. So there is a resynchronisation procedure: two platform engineers holding HSM quorum plus a DBA witness update the broker's database engine configuration, every step logged. It is tested as part of the broker-recovery criterion, because the realistic incident sequence is outage, then out-of-band intervention, then broker recovery, then resynchronisation — and the fourth step is the one that gets forgotten in the design and discovered at 3am.

**6. Audit as a design input, not an export.** Every issuance carries workload identity, requesting node as request metadata rather than as identity, purpose, TTL, and the approval basis — for automated issuance, the governing policy ID and version under which the workload was enrolled; for break-glass, the incident ticket and the dual authorisation. Streamed immutably to the SIEM on seven-year WORM retention. Audit records reference credential identifiers and lease IDs only. Credential material is never written to an audit log.

*Rejected:* retrofitting an audit trail after the broker is built. Expensive, and it produces gaps precisely where the interesting events were.

**7. Convert the DevOps lead rather than overrule him.** He keeps Kubernetes secrets — as a *consumption surface*. Pods want a file or an environment variable and he is not wrong about that. What he is wrong about is that base64 in etcd, readable by anyone with namespace `get`, with no read audit and no expiry, is a control. So the mount stays and the source changes: broker-injected, short TTL, rotated automatically. He loses nothing operationally, which is how an objection gets converted instead of suppressed.

**8. The Kubernetes pattern: injected, never stored in etcd.** GKE workloads run a broker agent sidecar that mounts credentials into a memory-backed volume at pod start. The Kubernetes Secret object holds only a broker token or OIDC JWT, never credential material. The sidecar fetches the real credential, renews in the background, and during a broker outage keeps serving the cached lease until it expires.

**Mechanism test.**

- *Who:* the DevOps lead configures sidecar injection through a mutating webhook.
- *Through what:* the agent sidecar plus an `emptyDir` volume with `medium: Memory`.
- *Enforced by:* an admission policy requiring the injection annotation on any pod in the payment namespace and denying pod specs that mount Secrets outside an allow-list, plus a scheduled scanner over Secret objects that alarms on any holding credential material.
- *Evidenced by:* the pod spec showing the memory mount and sidecar, the broker audit log showing lease request and renewal, and the scanner report showing zero credential-bearing Secrets in the payment namespace.

**9. Third-party API keys: custody, with an exit written down.** For vendor APIs that support neither dynamic credentials nor workload identity, the broker holds the key as a static secret and rotates it through the vendor's own rotation API. Each one is exception-registered with an owner, a review date and a plan to eliminate it — move to OAuth, negotiate dynamic credentials, or retire the integration.

**Mechanism test.**

- *Who:* a platform engineer configures the rotation schedule.
- *Through what:* a static secret plus a rotation job calling the vendor API.
- *Enforced by:* a quarterly CISO review of the exception register, escalating any key with no elimination plan.
- *Evidenced by:* the broker rotation log and the register entry with owner, review date and plan.

*Rejected:* treating third-party keys as "just another secret." That is how the custody bucket grows without limit while the metric that is supposed to measure it stays flat.

## Accepted degradation

A payment pod that crashes during a broker outage cannot get a new credential and therefore cannot restart. The payment service runs **N+2 warm capacity**, sized to absorb the loss of two pods across a four-hour outage without breaching throughput SLA. A pod that dies during an outage stays dead until the broker returns. This is written into the payment service SLO and approved by the payment service owner, because an accepted degradation that nobody signed is just an outage waiting for an owner.

*Rejected:* node-level memory storage for credential material so a pod restart survives the outage. That makes the credential node-scoped instead of workload-scoped, breaks the binding to workload identity, and lets any non-CDE pod scheduled onto that node read a CDE credential. Worse than the Kubernetes Secrets this design already rejected.

## The risk statement

A single compromised hardcoded credential in the payment path is a reportable breach with regulatory penalty, card-brand fines and mandatory notification.

**Derivation,** on the same method as Case 1. One credential against 150,000 card records at $5–$20 per record ($750K–$3M), plus card-brand fines ($50K–$500K), plus OCC or FDIC enforcement ($1M–$10M for wilful non-compliance).[^1] **$2M–$14M per breach event, midpoint around $8M.** The current state holds 4,000 such credentials, any one of which could trigger it.

The broker replaces them with short-lived, attributable, revocable credentials and an immutable audit trail, at a cost of a tier-0 commercial dependency — roughly $150K–$250K a year plus 1.5 FTE — against a seven-figure exposure that is currently unmeasured. The compensating control for the extended payment-path TTL is immediate revocation while the broker is healthy; the out-of-band runbook and the sealed break-glass provide kill capability when it is not.

## Reversal triggers

This design is superseded, not patched, if any of these is true by day 30.

1. **Payment consumers cannot be modified** (vendor black box, no release path) **and** Oracle does not support gradual password rollover. The design becomes file injection with atomic swap, and "no downtime" then rests on vendor behaviour nobody here controls.
2. **PassTickets are unavailable on the target LPAR** **and** the dual-RACF-userid fallback is not proven before the operator departs on day 20.
3. **Enterprise licensing is not approved by day 21.** The architecture reverts to evaluating open-source brokers with commercial support, or to a different shape entirely. An unsupported tier-0 dependency on the payment path is worse than what the bank has today.
4. **Measured DR recovery exceeds 4 hours and cannot be brought inside the 24-hour TTL ceiling.** Then the RTO gets fixed, or this architecture is superseded. The ceiling is not negotiable downward under schedule pressure, which is exactly when it would be.

## Acceptance criteria

One per credential class, each naming the artifact. All in a production-parity environment.

1. **Oracle.** A payment-adjacent database credential is rotated with transactions in flight and the old credential revoked immediately. Zero failed transactions, and an audit trail showing lease ID, workload identity and revocation timestamp. Evidence: broker audit log plus transaction monitor.
2. **Mainframe, PassTicket path** (conditional on assumption 3). A PassTicket is issued for a test transaction against the payment LPAR, and the old RACF password is verified non-functional after rotation. Evidence: CICS transaction log and RACF audit log.
3. **Mainframe, dual-userid fallback** (unconditional, before day 20). One userid rotates while the other serves traffic, the proxy target flips, a test transaction verifies it, and the rollback is proven. Evidence: proxy configuration log, CICS transaction log, RACF audit log.
4. **GCP, policy enforcement.** An attempted service account key creation is denied by organisation policy. Evidence: the denied `CreateKey` event in Cloud Audit Logs.
5. **GCP, key inventory.** A query across every service account in the organisation returns zero exported keys. Evidence: the timestamped inventory report.
6. **Kubernetes.** A pod starts with the agent sidecar, credential material exists only in the memory-backed mount and never in etcd, and a simulated broker outage leaves the sidecar serving its cached lease to expiry. Evidence: pod spec, broker lease log, and an etcd dump showing no credential material.
7. **Third-party API key.** A static key rotates through the vendor API and the exception register is updated with a new review date and a documented elimination plan. Evidence: broker rotation log and register entry.
8. **Broker outage.** Primary and replica are taken offline. The payment workload keeps processing on its cached 8-hour lease for at least four hours, new connections are blocked, a payment pod is killed during the outage and warm capacity absorbs it without SLA breach. The audit trail captures the issuance gap.
9. **Broker recovery after out-of-band intervention.** The broker is restored after a four-hour outage during which the Oracle payment account was repassworded out of band. The workload resumes renewal only after broker authority is resynchronised, with no process restart.
10. **DR promotion drill**, using production key holders and out-of-hours paging, to measure actual RTO. If measured RTO exceeds four hours, the payment-path TTL is raised before production cutover, capped at 24 hours. If it would need more than that, the RTO gets fixed or the architecture is superseded.
11. **Load test.** The broker sustains projected estate issuance — roughly 933 renewals a minute (4,000 on-premises leases on a 5-minute interval, 2,000 GCP leases on 15 minutes), with burst headroom to about 1,866. That is an upper bound assuming every lease renews at its shortest interval simultaneously. Renewal jitter is required to prevent a thundering herd after a partition, and the test proves it: cut the network for 60 seconds, restore, measure the spike.
12. **One payment-adjacent service** running in production on broker-injected credentials for at least two weeks without incident.

## First steps

Week one: verify all six assumptions (payment modifiability, Oracle patch level, PassTicket availability, interconnect headroom, regulator acceptance of the broker log as system of record, and broker budget). Open procurement. Document the mainframe operator's manual procedure and get a second pair of hands on it while he is still here. Run a secret scanner across every GitHub organisation to produce a verified inventory, expecting a material fraction of the 4,000 to be duplicates, dead code or decommissioned systems — that figure is a scanner output, not a reproducible count, and treating it as one is the first way this programme could lose credibility. Put the QSA review of the broker's PCI boundary on the calendar for week three.

[^1]: OCC, *In the Matter of Capital One, N.A.*, August 2020, an $80 million civil money penalty for deficiencies in cybersecurity and internal controls. The $1M–$10M range used here for a bank of roughly $2B in assets is extrapolated from larger enforcement actions rather than drawn from a comparable-size precedent, and the actual figure in any specific case turns on severity, duration and wilfulness.

---

*Previous: [Case 3, translating a zero trust mandate](case-3-zero-trust-acquisition.md). Method and grounding: [Design Cases](../decisions.md).*
