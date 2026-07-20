# Lanre Oluokun

**Senior Cloud Security Engineer** | CISSP, CCSP, CISM, ISSAP, GCP-PCA

Most security portfolios show you the final diagram. That's the least interesting part. The diagram tells you what someone built; it tells you nothing about what they rejected, what they got wrong, or what the design still cannot do.

So every project here ships with its architecture decision records. The rejected options are in there. So are the trade-offs I accepted and the assumptions I have not verified. In [BankVault](https://github.com/Bigbadlonewolf/bankvault) you can watch me reverse my own ADR-001 a week after writing it, because the condition I said would trigger a rethink actually triggered.

I work in GCP: identity, access, policy-as-code, and the compliance mapping that makes a control defensible to an auditor instead of merely green on a dashboard.

---

## What I do

- Cloud security engineering on GCP: IAM, Zero Trust access design, policy enforcement
- Architecture decision records, documenting the *why* rather than only the *what*
- GRC: PCI DSS, SOC 2, and NIST 800-53 mapped to technical controls
- Policy-as-Code with OPA/Rego, enforced in CI
- Compliance automation in GitHub Actions pipelines

---

## Certifications

- **CISSP**, Certified Information Systems Security Professional
- **CCSP**, Certified Cloud Security Professional
- **CISM**, Certified Information Security Manager
- **ISSAP**, Information Systems Security Architecture Professional
- **GCP-PCA**, Google Cloud Professional Cloud Architect

Digital badges available on request

---

## Projects

| Project | What it is | Status |
| --- | --- | --- |
| [**BankVault**](https://github.com/Bigbadlonewolf/bankvault) | Zero Trust just-in-time access broker for a loan origination workflow. GCP Privileged Access Manager issues a 30-minute IAM grant scoped by CEL to a single credit-report object. OIDC `max_age=0` forces a fresh MFA event on every request, and the broker denies outright when the identity provider is down. GLBA-scoped. | In progress. Architecture locked, ADR-001 through ADR-006 published |
| [**JIT Access Broker**](https://github.com/Bigbadlonewolf/JIT-ACCESS-BROKER) | Design-phase GitHub-native PAM policy engine (OPA/Rego). | Provisioning and revocation workflows in progress |
| [**Compliance as Code**](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE) | OPA/Rego policy checks against Terraform plans, mapped to PCI DSS v4.0, SOC 2 (CC6/CC7), and NIST 800-53. Runs in GitHub Actions with no cloud credentials required. | Shipped. 50/50 tests passing, five-job gated CI |
| [**GCP Hardened Landing Zone**](https://github.com/Bigbadlonewolf/GCP-HARDENED-LANDING-ZONE) | Terraform security baseline: CMEK, audit logging, VPC isolation, and OPA policy enforcement mapped to PCI DSS v4.0, NIST 800-53, and SOC 2. | Reference architecture. Not deployed to production |
| [**SecureVault**](https://github.com/Bigbadlonewolf/SecureVault) | GCP-native CSPM pipeline. Security Command Center findings route through Pub/Sub to a Cloud Function, which auto-remediates public buckets and open firewall rules and alerts on everything it does not recognize. | In progress |
| [**Vulnerability Management Program**](https://github.com/Bigbadlonewolf/Vulnerability-Management) | End-to-end vulnerability management on Azure: policy drafting, stakeholder buy-in, credentialed Tenable scanning, CAB process, full remediation cycle. 80% reduction in open findings. | Documented case study |
| [**Personal Site**](https://bigbadlonewolf.github.io/Lanreoluokun.com/) | ADRs, project write-ups, and design reasoning. Hugo, deployed on GitHub Pages. | [Live](https://bigbadlonewolf.github.io/Lanreoluokun.com/) |

Anything marked "in progress" still has a real ADR trail behind it. The reasoning is the deliverable as much as the code is.

---

## Background

I did not start in security.

Ten years in retail banking operations at First Bank of Nigeria taught me what a regulated environment demands of a control long before anyone writes Terraform for it. Before that, IT support at British American Tobacco in Ibadan, covering 150+ workstations on ServiceNow. In between, I founded and ran Bloominglo Limited, a logistics business with $500K+ annual revenue, which is where I learned what it costs when a process fails quietly.

Then I rebuilt in the US. I earned the CISSP, CCSP, CISM, and ISSAP while working, completed a GCP Cloud Security Architect engagement at ATBOD (2022 to 2023), and worked as a Cloud Vulnerability Engineer at LOG(N) Pacific (2023 to 2024). I am currently deepening hands-on GCP architecture practice through the Go Cloud Careers program while building the portfolio above.

That banking background is the reason I write ADRs the way I do. A control that cannot be explained to an examiner is a control that does not exist yet.

I am targeting Senior Cloud Security Engineer and Security Engineer roles at banks and fintechs. The architecture-level thinking on this page is the evidence.

---

## Contact

- [LinkedIn](https://www.linkedin.com/in/lanre-oluokun-04256040/)
- [X](https://twitter.com/IAMOLANREWAJU)
- [Personal site](https://bigbadlonewolf.github.io/Lanreoluokun.com/)
- [Email](mailto:lanre.oluokun@gmail.com)
