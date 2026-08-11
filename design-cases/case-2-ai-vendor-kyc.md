# Design Case 2: Approving an AI Vendor for KYC Without Losing the PII

*Part of [The decisions](../decisions.md). Rendered version on [my site](https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/case-2-ai-vendor-kyc/).*

## The scenario

The same bank, six months later. The onboarding team wants a third-party AI document-processing service for KYC: passports, utility bills, proof of address. Compliance is worried about a vendor touching customer PII and a model nobody inside the bank controls or can inspect. Leadership wants a decision and a design in 60 days.

## The decision, sentence one

**Approve, with conditions, and the conditions are the design.** Defined PII protection. Retention and deletion after processing. No training on bank data. Subprocessor disclosure and approval. Breach notification inside the regulatory window. Continuous audit rights.

If the vendor will not sign those, the recommendation flips to reject. A condition with no consequence attached is a wish. The willingness to say no is the only thing that makes the yes credible.

## Scope

**In:** the complete lifecycle of the KYC document and the vendor's output. Capture in the bank's channels, transmission to the vendor, processing on vendor infrastructure, retention and deletion on both sides, and the onboarding decision path the model's verdict feeds.

**Out:** rebuilding the onboarding workflow. Any use of this vendor beyond KYC documents. Expansion is a future decision with its own assessment, not a rider on this one.

## Constraints

- **60 days.** Due diligence runs in parallel with the design, not before it.
- **Onboarding latency budget.** The check sits inside the customer acquisition path. Added friction breaks conversion.
- **Black-box model.** Internals cannot be inspected, so the design cannot rest on understanding the model. It has to bound what the model can see and what its output is allowed to do.
- **Unknown training data.** Provenance and bias questions cannot be answered directly. They get bounded contractually and monitored empirically.
- **Political reality.** Leadership already wants this vendor. The job is to make yes safe, not to say no from the sidelines.

## Trust boundaries

1. **Capture channel to bank environment.** Standard. The bank controls it.
2. **Bank to vendor.** The decisive one. The vendor has to read cleartext documents in order to process them, so from this line onward technical controls effectively end and the control plane becomes contractual and procedural. Encryption protects data in motion and at rest. It cannot protect data being processed. Any design that answers this boundary with encryption alone has not understood what processing means.
3. **Vendor to its subprocessors.** If the service wraps a third-party foundation model, customer PII may reach a fourth party the bank never assessed. Subprocessor disclosure and approval rights are not boilerplate here. They are a trust boundary made visible.
4. **The decision path.** Where does the model's output land? If it influences an onboarding accept or decline, fair-lending and adverse-action obligations attach to a model nobody can fully explain. This boundary sets the bank's regulatory exposure more than any technical one does.

```mermaid
flowchart LR
    subgraph BANK["Bank environment"]
        C["Capture channel<br>the bank controls it"] --> B["Bank systems<br>minimum image set"]
    end

    subgraph VENDOR["Vendor"]
        AI["AI document processing<br>reads cleartext to work"]
    end

    subgraph SUB["Subprocessor"]
        FM["Foundation model<br>a possible 4th party"]
    end

    B -- "BOUNDARY 2 - technical controls end,<br>contractual controls begin" --> AI
    AI -- "BOUNDARY 3" --> FM

    AI --> MO["Model output<br>advisory only"]
    MO --> H["Human decision<br>or a deterministic rule"]
    H --> OD["Onboarding decision<br>accept or decline"]
    OD --> FL["BOUNDARY 4<br>fair lending and adverse action<br>obligations attach here"]

    B -.-> TECH["What the bank can enforce<br>TLS in transit - customer-managed keys<br>minimum image set - every transmission logged"]
    AI -.-> CON["What the bank can only agree<br>retention and deletion SLA - no training on bank data<br>subprocessor approval - breach notification - audit rights"]

    classDef key fill:#F7C948,stroke:#EAB938,color:#1A1A1A
    class AI,FL key
```

*Read the two dotted boxes rather than the flow. **Everything the bank can
enforce sits before boundary two; everything after it is something the bank can
only agree.** That is why the paperwork is the primary control here and not an
administrative step that happens once the architecture is finished.*

## Assumptions

1. **The vendor will sign the data-handling commitments.** If false, the recommendation flips to reject, and leadership hears it in those words.
2. **The model's output is advisory.** A human or a deterministic rule makes the actual onboarding decision. If the business wants straight-through automated decisions, the regulatory exposure changes category and the design changes with it.

## Design commitments

**1. Contractual control plane, carrying the primary load.** A data processing agreement specifying retention and deletion SLAs after processing, a prohibition on training vendor models with bank data, subprocessor disclosure with approval rights, breach notification inside the bank's regulatory window, and audit rights (or a current SOC 2 Type II plus penetration-test attestation standing in for them).

This is the uncomfortable part of the design, and it is where the real work sits. Boundary two cannot be defended with technology, so the paperwork *is* the control. Treating it as procurement admin is how banks end up with a technical architecture that stops precisely where the risk starts.

**2. Technical minimization, carrying what it can.** Transmit the minimum image set the check requires, not documents plus metadata plus account context, over TLS, to a regional processing endpoint, with customer-managed encryption keys where the vendor supports them. Log every document transmitted so the bank can prove exactly what crossed boundary two. Retain in the bank's own environment whatever AML record-keeping rules require.

A correction worth stating plainly, because the instinct runs the other way: **KYC data is not anonymized.** Customer identification programs exist to verify identity, and the underlying records must be retained by law. Anonymizing KYC data would be a compliance violation wearing the costume of a control. Minimization governs what the vendor sees and keeps. Retention obligations govern what the bank must hold. Confusing the two fails in both directions at once.

```mermaid
flowchart TB
    K["KYC document<br>passport, proof of address"]

    K -- "minimisation governs<br>what leaves" --> OUT["What crosses to the vendor<br>the minimum image set the check requires<br><br>NOT SENT: account context, metadata,<br>unrelated documents"]
    K -- "retention governs<br>what is kept" --> KEEP["What the bank must retain<br>identity records, for as long as the rules require<br><br>AND NOT ANONYMISED: identification<br>programmes exist to verify identity"]

    OUT --> NOTE["Anonymising KYC data would be a compliance violation<br>wearing the costume of a control.<br>Confusing the two rules fails in both directions."]
    KEEP --> NOTE

    classDef key fill:#F7C948,stroke:#EAB938,color:#1A1A1A
    class K,NOTE key
```

*Two rules pointing away from each other, applied to one document. The failure
this exists to prevent is the reflex answer in an interview, where **anonymise
the data** sounds like the responsible thing to say right up until someone asks
how an identity check works on a record that no longer identifies anyone.*

**3. Model governance, monitored on two separate axes.** The vendor's model gets the same discipline regulators apply to any consequential model: documented validation, ongoing monitoring, and a challenger comparison on a sample of decisions. Two different failures hide behind the word "drift," and one control cannot catch both.

*Regression testing, which catches the vendor silently changing the model.*

- *Who:* the Model Risk Committee, monthly.
- *Through what:* a held-out labelled validation set of 1,000 documents, baselined at 95% agreement with human review.
- *Enforced by:* weekly accuracy measurement, with a 2 percentage point decline from baseline triggering revalidation and vendor escalation.
- *Evidenced by:* the model governance register, holding the validation report, the drift log and the challenger comparison.

*Input drift monitoring, which catches the real world changing underneath a model that has not.*

- *Who:* the operations team, sampling 100 production documents a month for human adjudication.
- *Through what:* side-by-side comparison of vendor output against human review on the sample.
- *Enforced by:* a 5 percentage point decline in agreement from the held-out baseline triggering investigation and vendor escalation.
- *Evidenced by:* the monthly sampling report, with adjudication log and trend.

The exposure analysis, covering fair-lending risk, adverse-action reasoning and model risk management expectations, is written up as a decision record before the vendor goes live, so compliance review starts from an artifact rather than a promise to produce one.

*Rejected:* building document processing in-house. In 60 days the bank produces a weaker model with no validation history. The vendor's risk is knowable and contractible. An in-house build's risk is neither.

*Also rejected:* pre-anonymizing the documents. Technically broken for image processing, and legally wrong for identity records.

**4. Decision-path control: advisory only, with the rules on the bank's side.** The vendor's output feeds a bank-side deterministic rule engine. There is no path from vendor output to an automated decline. Adverse-action reasoning is generated from the bank's own rules, never from model output, and a human or a deterministic rule makes the final call.

**Mechanism test.**

- *Who:* the onboarding business owner owns the rule set.
- *Through what:* a bank-side rule engine consuming only schema-validated vendor extractions.
- *Enforced by:* code review and required status checks on the rules repository, with no production deploy without dual approval.
- *Evidenced by:* the rule version log and a decision audit trail showing, for each decision, the vendor output (confidence and extracted fields) alongside the bank-side rule that actually fired.

*Rejected:* straight-through processing, where vendor output triggers accept or decline directly. That attaches fair-lending and adverse-action obligations to a black-box model the bank cannot explain, which is the one exposure this design exists to avoid.

**5. Treating the document as hostile input.** Customer-uploaded images are untrusted. If the vendor's pipeline is vision-model based, the document is untrusted input to a model whose output the bank then consumes — which makes a crafted document a path from a stranger's upload into the bank's decision systems.

**Mechanism test.**

- *Who:* a security engineer owns the output schema and the validation rules.
- *Through what:* a bank-side validation layer sitting between the vendor API and anything downstream.
- *Enforced by:* schema validation admitting expected fields only (name, address, date, document type), a minimum confidence threshold, and an out-of-distribution alarm on field-count mismatch, format deviation or unexpected narrative content.
- *Evidenced by:* the rejection log, holding quarantined documents and alarm timestamps, reviewed quarterly for false-positive rate.

*Rejected:* consuming vendor narrative output or free-text interpretation without validation. That is the injection path, left open.

## The risk statement

Vendor retention or leakage of onboarding PII means regulatory penalty, mandatory notification and customer attrition.

**Derivation,** on the same method as Case 1. State breach notification at $5–$20 per record across 250,000 customers ($1.25M–$5M), plus OCC or FDIC enforcement for a GLBA violation ($1M–$10M),[^1] plus attrition. Realistically a **seven-figure to low-eight-figure event riding on a five-figure annual contract**. The conditions in this design are cheap insurance against the plausible version of that.

What the board gets offered is **reasonable assurance**. No honest officer promises full assurance over a third-party black box, and the difference between those two phrases is whether a risk statement means anything at all.

## Reversal triggers

This design is superseded, not patched, if any of these is true by day 30.

1. **The vendor will not sign the data-handling commitments.** The recommendation flips to reject and no integration proceeds.
2. **The business insists on straight-through automated decisions.** The exposure changes category — fair lending, adverse action, model risk management — and needs a separate 90-day assessment. The 60-day scope is abandoned rather than stretched.
3. **The vendor uses undisclosed subprocessors.** Disclosure is a hard requirement. Without it the contractual control plane is void, and the contractual control plane is the design.

## Acceptance criteria

1. **The contract gate.** A signed DPA carrying every required clause: retention, deletion, no training, subprocessor approval, breach notification, audit rights. Evidence: the executed contract and the legal review checklist.
2. **Schema validation bites.** A crafted document carrying a prompt-injection attempt is submitted and rejected by the validation layer before anything downstream sees it. Evidence: the rejection log with alarm timestamp.
3. **Drift triggers action, not a note.** A 2-point accuracy decline on the validation set triggers the revalidation workflow and vendor escalation. Evidence: Model Risk Committee minutes and the vendor's response.
4. **The decision path holds.** A sample of 100 onboarding decisions shows, for each, the vendor output and the bank-side rule that fired, with zero decisions made by vendor output alone. Evidence: the decision audit trail.

## First steps

Week one: verify both assumptions, redline the data processing agreement, get the subprocessor list, agree the decision-path constraint (advisory only) with the onboarding business owner, and define the output schema and validation rules for the injection control.

[^1]: OCC, *In the Matter of Capital One, N.A.*, August 2020, an $80 million civil money penalty for deficiencies in cybersecurity and internal controls. The $1M–$10M range used here for a bank of roughly $2B in assets is extrapolated from larger enforcement actions rather than drawn from a comparable-size precedent, and the actual figure in any specific case turns on severity, duration and wilfulness.

---

*Previous: [Case 1, the payment API migration](case-1-payment-api-migration.md). Next: [Case 3, translating a zero trust mandate](case-3-zero-trust-acquisition.md).*
