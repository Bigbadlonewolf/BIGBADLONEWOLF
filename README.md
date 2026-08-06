# Lanre Oluokun

**Enterprise Security Architect** | CISSP, CCSP, CISM, ISSAP, GCP-PCA

Most security portfolios show you the final diagram. That is the least interesting part. A diagram tells you what someone built. It tells you nothing about what they rejected, what they got wrong, or what the design still cannot do.

So every project here ships with its architecture decision records. The rejected options are in there. So are the trade-offs I accepted and the assumptions I have not verified. In BankVault you can watch me reverse my own ADR-001 seven days after writing it, because the condition I named as a trigger for a rethink actually fired.

I work in GCP: identity, access, policy-as-code, and the compliance mapping that makes a control defensible to an examiner instead of merely green on a dashboard.

**The implementation repositories are private. The reasoning is not.** Every architecture decision record below is published in full on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/). That is the artifact worth reading anyway.

---

## How I work

An architect is judged on the decisions, not the deliverables. Three habits I can show evidence for:

**I name the condition that would kill my own decision.** ADR-001 for BankVault chose a custom just-in-time access broker over Google's Privileged Access Manager, which was in Preview at the time. The ADR recorded one exit condition: re-evaluate at general availability. PAM reached GA. I ran the re-evaluation, it told me to delete code I had just written, and ADR-005 replaced the grant-issuance mechanism. ADR-001 carries a supersession note and a table of which sections stand and which fell.

**I put my designs under adversarial review and publish what it finds.** The compliance-as-code repository has an audit log recording three external review rounds with 37 documented findings. Round one found an encryption check that passed when the field was `null`. Round two found a bug I introduced myself while fixing a different one, which is written up rather than quietly reverted.

**I write down what a control cannot do.** Every project has a limits section. A policy engine reading Terraform plan output can enforce "you must declare a value." It cannot enforce "that value is truthful," because verifying that needs runtime knowledge the plan JSON does not contain. That gap belongs in the documentation, not behind a weaker workaround.

---

## Selected work

| Project | The architecture problem | State |
| --- | --- | --- |
| [**Compliance as Code**](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE) | One control, three frameworks, three copies of the detection logic that silently drifted apart. Detection now lives once in `policies/controls/`; PCI DSS v4.0, SOC 2 and NIST 800-53 packages attach citations only. | Public. 163/163 OPA tests, five gated CI jobs, [ADR-001](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE/blob/main/docs/adr/001-two-rail-control-engine.md) published |
| **BankVault** | Standing access survives quarterly review. An underwriter gets 30 minutes against one credit report, scoped by IAM Condition, gated on a fresh MFA event, with the broker denying outright when the identity provider is down. GLBA-scoped. | Private repo. Six ADRs, [published in full](https://bigbadlonewolf.github.io/Lanreoluokun.com/posts/) |
| **SecureVault** | Security Command Center findings pile up unread. Event-driven pipeline routes them through Pub/Sub to a Cloud Function that auto-remediates two finding types, escalates the rest, and refuses to act on anything it does not recognise. | Private repo. Nine ADRs, [eight published](https://bigbadlonewolf.github.io/Lanreoluokun.com/adrs/) |
| **GCP Hardened Landing Zone** | Terraform security baseline across four layers: CMEK, audit logging, VPC isolation, OPA enforcement. | Private repo. Reference architecture, not deployed |
| **JIT Access Broker** | GitHub-native PAM policy engine in OPA/Rego. | Private repo. Policy engine built, provisioning and revocation in progress |
| [**Vulnerability Management**](https://github.com/Bigbadlonewolf/Vulnerability-Management) | Guided walkthrough of Josh Madakor's public Azure lab: policy drafting, stakeholder buy-in, credentialed Tenable scanning, CAB process, remediation cycle. | Public. Completed course lab, attributed |
| [**Personal Site**](https://bigbadlonewolf.github.io/Lanreoluokun.com/) | Where the architecture decision records live: every ADR in full text, project write-ups, and the design reasoning behind each one. Hugo, deployed on GitHub Pages. | [**Live**](https://bigbadlonewolf.github.io/Lanreoluokun.com/) |

Anything marked private is available on request. The decision trail behind each one is public regardless, because the reasoning is the deliverable as much as the code is.

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
