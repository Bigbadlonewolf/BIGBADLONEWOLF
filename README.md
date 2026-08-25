# Lanre Oluokun

**Enterprise Security Architect** | CISSP, CCSP, CISM, ISSAP, GCP-PCA

Most security portfolios show you the final diagram. That is the least interesting part. A diagram tells you what someone built. It tells you nothing about what they rejected, what they got wrong, or what the design still cannot do.

So every project here ships with its architecture decision records. The rejected options are in there. So are the trade-offs I accepted and the assumptions I have not verified. In BankVault I reversed my own ADR-001 seven days after writing it, because the condition I had named as the trigger for a rethink actually fired.

I work in GCP: identity, access, policy-as-code, and the compliance mapping that makes a control defensible to an examiner instead of merely green on a dashboard.

**The decisions are the deliverable, not the diagrams.** Every build listed below is open, its ADRs included, and [the four design cases](decisions.md) are published in full right here. Read the decision records before the code — they are the artifact worth your time.

---

## How I work

An architect is judged on the decisions, not the deliverables. Three habits I can show evidence for:

**I name the condition that would kill my own decision.** ADR-001 for BankVault chose a custom just-in-time access broker over Google's Privileged Access Manager, which was in Preview at the time. The ADR recorded one exit condition: re-evaluate at general availability. PAM reached GA. I ran the re-evaluation, it told me to delete code I had just written, and ADR-005 replaced the grant-issuance mechanism. ADR-001 carries a supersession note and a table of which sections stand and which fell.

**I put my designs under adversarial review and publish what it finds.** The compliance-as-code repository has an audit log recording three external review rounds with 37 documented findings. Round one found an encryption check that passed when the field was `null`. Round two found a bug I introduced myself while fixing a different one, which is written up rather than quietly reverted.

**I write down what a control cannot do.** Every project has a limits section. A policy engine reading Terraform plan output can enforce "you must declare a value." It cannot enforce "that value is truthful," because verifying that needs runtime knowledge the plan JSON does not contain. That gap belongs in the documentation, not behind a weaker workaround.

---

## The decisions

Four scenarios for one regional bank, worked end to end. Each frames the problem before naming a component, and every commitment carries the alternative I rejected. An answer with no rejected alternative is a preference, not a decision. One bank profile runs through all four, so the money figures are comparable rather than invented per case.

| Case | The decision | Rejected |
| --- | --- | --- |
| [Migrate the payment API in 90 days, regulator watching](design-cases/case-1-payment-api-migration.md) | Tokenize at the payment service provider, so no card number reaches bank infrastructure and the cardholder data environment collapses to the integration boundary | An in-house token vault, which hands a three-person team a regulated environment to run |
| [Approving an AI vendor for KYC without losing the PII](design-cases/case-2-ai-vendor-kyc.md) | Approve with conditions, because past the vendor boundary the control plane is contractual rather than technical | Pre-anonymizing the documents, which is broken for image processing and wrong in law for records the bank must retain |
| [The board wants zero trust, and that is not a design](design-cases/case-3-zero-trust-acquisition.md) | A phased migration of trust decisions from network location to identity, expressed as four testable properties | Buying a platform first, which automates the confusion at scale before identity is unified |
| [Four thousand hardcoded credentials, and a payment pipeline that cannot stop](design-cases/case-4-secrets-credential-rotation.md) | One HSM-backed credential broker as the issuance plane, with the payment path failing static rather than closed so an outage blocks new issuance without stopping in-flight processing | Per-platform secret stores, cheaper up front and producing four partial audit logs where the regulator asked for one system of record |

Case 1 also carries the decision I got wrong first. The access broker had its own revocation function and I deleted it, because Privileged Access Manager expires the grant on its own. What replaced it detects an overrun and does not contain one, so the defensible claim is "detected inside roughly one sweep," not "contained in fifteen minutes."

Case 4 carries the figure I am least comfortable with and say so on the page: an eight-hour credential on the payment path, sized at four hours of unvalidated recovery target plus four hours of margin, with a twenty-four hour ceiling above which the recovery time gets fixed instead of the lease extended.

**[Read the method and all four cases](decisions.md)**, or the [rendered versions](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/) on my site.

---

## Selected work

| Project | The architecture problem | State |
| --- | --- | --- |
| [**Compliance as Code**](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE) | One control, three frameworks, three copies of the detection logic that silently drifted apart. Detection now lives once in `policies/controls/`; PCI DSS v4.0, SOC 2 and NIST 800-53 packages attach citations only. | Public. 163/163 OPA tests, five gated CI jobs, [ADR-001](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE/blob/main/docs/adr/001-two-rail-control-engine.md) published |
| [**BankVault**](https://github.com/Bigbadlonewolf/bankvault) | Standing access survives quarterly review. An underwriter gets 30 minutes against one credit report, scoped by a PAM entitlement with an object-prefix condition, gated on a fresh MFA event, with the broker denying outright when the identity provider is down. GLBA-scoped. | Public. Deployed to GCP — 43 resources, both PAM entitlements live. Broker fails closed with no IdP wired; six ADRs, two of them reversals |
| [**SecureVault**](https://github.com/Bigbadlonewolf/SecureVault) | Security Command Center findings pile up unread. Event-driven pipeline routes them through Pub/Sub to a Cloud Function that auto-remediates two finding types, escalates the rest, and refuses to act on anything it does not recognise. | Public. Deployed and verified end to end — firewall remediation, audit trail, delivered alert, all reproducible per [VERIFICATION.md](https://github.com/Bigbadlonewolf/SecureVault/blob/main/docs/VERIFICATION.md). Ten ADRs |
| [**GCP Hardened Landing Zone**](https://github.com/Bigbadlonewolf/GCP-HARDENED-LANDING-ZONE) | Terraform security baseline across four layers: CMEK, audit logging, VPC isolation, OPA enforcement. | Public. Reference architecture, not deployed |
| [**Vulnerability Management**](https://github.com/Bigbadlonewolf/Vulnerability-Management) | Guided walkthrough of Josh Madakor's public Azure lab: policy drafting, stakeholder buy-in, credentialed Tenable scanning, CAB process, remediation cycle. | Public. Completed course lab, attributed |
| [**Personal Site**](https://bigbadlonewolf.github.io/Lanreoluokun.com/) | The four design cases in full, the seven-step method behind them, and the career arc. Hugo, deployed on GitHub Pages. | [**Live**](https://bigbadlonewolf.github.io/Lanreoluokun.com/) |

Every repository above is open, decision records included, because the reasoning is the deliverable as much as the code is.

---

## Certifications

- **CISSP**, Certified Information Systems Security Professional
- **CCSP**, Certified Cloud Security Professional
- **CISM**, Certified Information Security Manager
- **ISSAP**, Information Systems Security Architecture Professional
- **GCP-PCA**, Google Cloud Professional Cloud Architect

Digital badges available on request.

---

## Background

I did not start in security.

Ten years in retail banking operations at First Bank of Nigeria taught me what a regulated environment demands of a control long before anyone writes Terraform for it. Before that, IT support at British American Tobacco in Ibadan, covering 150+ workstations on ServiceNow. In between I founded and ran Bloominglo Limited, a logistics business with $500K+ annual revenue, which is where I learned what it costs when a process fails quietly.

Then I rebuilt in the US. I earned the CISSP, CCSP, CISM and ISSAP while working. My architecture-titled experience is one year as Cloud Security Architect at ATBOD (Sep 2022 to Aug 2023), followed by a year as Cloud Vulnerability Engineer at LOG(N) Pacific (2023 to 2024). Since September 2024 I have been in the Go Cloud Careers Executive Architect Program, a scenario-based architecture training program, building the portfolio above alongside it.

That banking background is why I write ADRs the way I do. A control that cannot be explained to an examiner is a control that does not exist yet.

I am targeting Security Architect and Cloud Security Architect roles at banks, fintechs and regulated financial institutions. The decision trail on this page is the evidence.

---

## Contact

- [LinkedIn](https://www.linkedin.com/in/lanre-oluokun-04256040/)
- [Personal site](https://bigbadlonewolf.github.io/Lanreoluokun.com/)
- [X](https://twitter.com/IAMOLANREWAJU)
- [Email](mailto:lanreolu88@gmail.com)
