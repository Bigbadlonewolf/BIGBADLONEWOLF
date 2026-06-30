<div align="center">

# Lanre Oluokun

### Cloud Security Architect · CISSP · CCSP · CISM · ISSAP · GCP-PCA

*Security-by-design for financial services. Architecture decisions, compliance automation, and cloud-native controls.*

[Portfolio](https://bigbadlonewolf.github.io/Lanreoluokun.com/) · [LinkedIn](https://www.linkedin.com/in/lanre-oluokun-04256040/) · [ADRs](https://bigbadlonewolf.github.io/Lanreoluokun.com/)

---

</div>

## What I Do

I design and implement security architectures for cloud-native financial workloads. My work lives at the intersection of **security engineering**, **compliance automation**, and **risk architecture** — translating regulatory requirements into deployed controls, not slide decks.

Currently contracted as an Executive Architect at Go Cloud Careers. Targeting Security Architect and Executive Architect roles at financial institutions.

## Credentials

| Certification | Focus Area |
|---|---|
| **CISSP** | Enterprise security strategy & governance |
| **CCSP** | Cloud security architecture (AWS/GCP/Azure) |
| **CISM** | Information risk management & incident response |
| **ISSAP** | Security architecture & design |
| **GCP-PCA** | Google Cloud platform architecture |

## Featured Projects

### [Compliance as Code](https://github.com/Bigbadlonewolf/COMPLIANCE_AS_CODE)
OPA/Rego policy enforcement mapped to PCI DSS v4.0, SOC 2, and NIST 800-53. 50/50 passing unit tests. Five-job gated CI pipeline that blocks noncompliant Terraform plans before they hit production.

**Key:** `OPA/Rego` · `Terraform` · `GitHub Actions` · `PCI DSS` · `SOC 2` · `NIST 800-53`

### [GCP Hardened Landing Zone](https://github.com/Bigbadlonewolf/GCP-HARDENED-LANDING-ZONE)
Zero Trust reference architecture on GCP — 4-layer build with IAP, Istio mTLS, OPA default-deny policies, and Terraform automation. Designed for financial workload isolation and regulatory compliance.

**Key:** `GCP` · `IAP` · `Istio` · `mTLS` · `Terraform` · `Zero Trust`

### [SecureVault](https://github.com/Bigbadlonewolf/SecureVault)
Event-driven GCP security findings pipeline: SCC Security Health Analytics → Pub/Sub → Cloud Function → email alerts. Covers PUBLIC_BUCKET_ACL, OPEN_FIREWALL, OVER_PRIVILEGED_SERVICE_ACCOUNT. Runs on free tier.

**Key:** `GCP` · `SCC` · `Pub/Sub` · `Cloud Functions` · `Security Monitoring`

### [JIT Access Broker](https://github.com/Bigbadlonewolf/JIT-ACCESS-BROKER)
GitHub-native Privileged Access Management with OPA/Rego policy enforcement, Terraform provisioning, and automatic privilege revocation. Just-in-time access without a separate PAM vendor.

**Key:** `GitHub Actions` · `OPA/Rego` · `Terraform` · `PAM` · `Zero Standing Privileges`

### [Vulnerability Management Program](https://github.com/Bigbadlonewolf/Vulnerability-Management)
End-to-end vulnerability management lifecycle: policy drafting, stakeholder buy-in, credentialed Tenable scanning, CAB remediation cycle. 80% vulnerability reduction in first cycle.

**Key:** `Azure` · `Tenable` · `PowerShell` · `CAB Process` · `Risk Management`

## Architecture Decision Records

I document decisions *before* the meeting, not after. Current ADRs cover:

- **ADR-001:** AI Vendor Deployment — ECOA/Reg B and SR 11-7/OCC 2011-12 compliance exposure analysis
- **ADR-002:** SecureVault GCP Pipeline — Event-driven vs. scheduled scanning architecture
- **ADR-003:** ZTNA Layer 4 Policy Enforcement — OPA default-deny vs. network-level segmentation

[Read all ADRs →](https://bigbadlonewolf.github.io/Lanreoluokun.com/)

## Architecture Philosophy

> **"No security assumptions left to the individual engineer."**

Every control I design is automated, tested, and enforced before deployment. Manual checklists are a failure mode. Compliance is a property of the system, not a quarterly checkbox.

**Principles I operate by:**
- **Preventive over detective** — Block violations at the PR, not in the audit
- **Traceable over plausible** — Every control maps to a regulation; every policy has a test
- **Default-deny over implied-trust** — No access without explicit justification
- **Cost-aware over cost-unlimited** — Effective security at $0–5/month where possible

## Background

Ten years in retail banking at First Bank of Nigeria. Rebuilt from the ground up in the US — security guard to cloud security architect. Entrepreneurial operator (ran Bloominglo Limited, ~$650K annual turnover). Now building at the architecture level.

## Let's Connect

- **[Portfolio & ADRs](https://bigbadlonewolf.github.io/Lanreoluokun.com/)** — Architecture decisions and project write-ups
- **[LinkedIn](https://www.linkedin.com/in/lanre-oluokun-04256040/)** — Professional network
- **Email:** lanre.okunola@gmail.com

---

*Currently targeting Security Architect and Executive Architect roles at financial institutions. Open to fractional CISO advisory engagements.*
