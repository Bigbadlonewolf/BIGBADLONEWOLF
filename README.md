# Lanre Oluokun

Senior Cloud Security Engineer | CISSP, CCSP, CISM, ISSAP, GCP-PCA

Ten years in infrastructure and security before moving into cloud-native security engineering — retail banking operations, IT infrastructure support, and a GCP Cloud Security Architect engagement at ATBOD (2022–2023). I design like an architect and ship like an engineer: every project below includes the ADR trail showing the actual tradeoffs, not just the final diagram.

---

## What I do

- Cloud security engineering on GCP: IAM, Zero Trust access design, policy enforcement
- Architecture decision records (ADRs) — documenting *why*, not just *what*
- GRC: PCI DSS, SOC 2, NIST 800-53 mapped to technical controls
- Policy-as-Code with OPA/Rego, enforced in CI
- Compliance automation in GitHub Actions / CI pipelines

---

## Certifications

- CISSP
- CCSP
- CISM
- ISSAP
- Google Cloud Professional Cloud Architect

---

## Projects

| Project | What it is | Status |
|---|---|---|
| **BankVault** | GCP Zero Trust JIT access broker for a loan origination workflow. PAM-based time-bound IAM grants, OIDC `auth_time`/`max_age=0` for MFA freshness validation, scoped to GLBA/NPI compliance. ADR-001 documents the architecture decisions and alternatives considered. | In progress — architecture locked, ADR-001 published |
| **COMPLIANCE_AS_CODE** | OPA/Rego policy checks against Terraform plans, mapped to PCI DSS v4.0, SOC 2, NIST 800-53. Runs in GitHub Actions with no cloud credentials required. | Deployed — 50/50 passing tests, 5-job gated CI |
| **ZTNA Reference Architecture** | 4-layer Zero Trust design for GCP: IAP, Istio mTLS, OPA default-deny, Terraform. | Designed — scaffolded, pending live org provisioning |
| **Vulnerability Management Program** | End-to-end vulnerability management on Azure: Tenable credentialed scanning, CAB remediation cycle, 80% reduction in open findings. | Documented case study |
| **SecureVault** | GCP-native CSPM concept: SCC findings routed through Pub/Sub to a Cloud Function for alerting on public buckets, open firewalls, and over-privileged service accounts. | In progress |
| **Personal Site** | ADRs, project write-ups, and security design reasoning. Hugo + GitHub Pages. | [Live](https://bigbadlonewolf.github.io/Lanreoluokun.com/) |

Every "in progress" or "designed" project has a real ADR behind it — the reasoning is the deliverable as much as the code.

---

## Background

Ten years in infrastructure, banking, and security operations. Started in IT support at British American Tobacco in Ibadan, Nigeria (150+ workstations, ServiceNow). Ten-plus years in retail banking operations at First Bank of Nigeria. Founded and ran Bloominglo Limited, a logistics business (~$650K annual turnover). Rebuilt in the US, earning CISSP, CCSP, CISM, and ISSAP while working, then completed a GCP Cloud Security Architect engagement at ATBOD (2022–2023) and a Cloud Vulnerability Engineer role at LOG(N) Pacific (2023–2024). Currently deepening hands-on GCP architecture practice through an intensive cloud security program (Go Cloud Careers) while building the portfolio above.

Targeting Senior Cloud Security Engineer / Security Engineer roles at banks and fintechs — with the architecture-level thinking on this page as the evidence for where I'm headed next.

---

## Contact

- [LinkedIn](https://www.linkedin.com/in/lanre-oluokun-04256040/)
- [Personal site](https://bigbadlonewolf.github.io/Lanreoluokun.com/)
- [Email](mailto:lanre.oluokun@gmail.com)
