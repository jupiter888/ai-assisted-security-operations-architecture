# Project 7 — STRIDE Threat Model

## 1. Purpose and Scope

This threat model evaluates security risks associated with processing untrusted security-event logs through deterministic detection, local AI-assisted triage, security screening, and simulated human approval controls.

The primary security objectives are to preserve evidence integrity, prevent unauthorized operational actions, identify selected model-output contamination patterns, and maintain human review boundaries.

The laboratory uses synthetic data and local, authorized systems.

**Status:** Implemented and partially validated. Residual risks remain. Not production-ready.

## 2. Protected Assets

- Integrity of synthetic security-event records and ground-truth labels.
- Integrity of deterministic detection rules and alert results.
- Confidentiality of security-event data and model inputs.
- Integrity of AI-generated findings, priorities, and supporting references.
- Integrity of evaluation metrics, reports, and test evidence.
- Integrity of reviewer identity and approval records.
- Human authority over security decisions and response authorization.
- Availability of the event-processing and model-inference pipeline.

## 3. Threat Actors and Capabilities

### TA-01 — Malicious Log Contributor

Can control selected fields of synthetic security events, including message text.

May insert instructions, false classification claims, or unsupported authorization assertions.

Cannot legitimately modify trusted system instructions, detection rules, or ground-truth labels.

### TA-02 — Unauthorized Local Caller

Can attempt to invoke laboratory functions with malformed input, fabricated approval objects, or asserted reviewer identities.

Does not possess independently verified administrative authority.

### TA-03 — Compromised or Misconfigured Component

Represents simulated failures of event integrity, detection configuration, evidence handling, or approval processing.

No external systems are compromised by this laboratory.

### TA-04 — Model-Output Manipulator

Attempts to influence AI-generated findings through attacker-controlled event content.

This actor may overlap with TA-01 and does not imply direct access to model weights or trusted instructions.

## 4. Trust Boundaries

The boundaries align with `docs/architecture/system-architecture.md`.

| Boundary | Description |
|---|---|
| TB-01 | Event source to event validation |
| TB-02 | Validated events to language model |
| TB-03 | Language model to finding validation |
| TB-04 | Model findings to security screening |
| TB-05 | Triage output to human analyst |
| TB-06 | Reviewer identity to approval registry |
| TB-07 | Approval registry to action authorization |

## 5. STRIDE Threat Register

| ID | Category | Threat scenario | Boundary | Mitigation and status |
|---|---|---|---|---|
| P7-T01 | Spoofing | Event falsely claims trusted source identity | TB-01 | Schema validation implemented; authenticated provenance not implemented |
| P7-T02 | Tampering | Event fields or labels altered to manipulate results | TB-01 | Separate synthetic fixtures and evaluation outputs; cryptographic integrity not established |
| P7-T03 | Repudiation | Finding cannot be traced to events or rules | TB-03/TB-05 | Event IDs, rule IDs, model outputs, and evaluation evidence recorded; durable audit logging not established |
| P7-T04 | Information Disclosure | Logs or findings expose sensitive information | TB-02/TB-05 | Synthetic data used; production redaction and access controls not validated |
| P7-T05 | Denial of Service | Malformed or oversized batches exhaust processing | TB-01/TB-02 | Schema and batch limits implemented; broader resource-exhaustion testing limited |
| P7-T06 | Elevation of Privilege | Log instructions attempt unauthorized response actions | TB-02/TB-07 | No autonomous response execution in evaluated workflow; prompt injection remains possible |
| P7-T07 | Tampering | Untrusted log instructions manipulate findings | TB-02/TB-04 | Input instruction and output claim screening implemented and tested; incomplete detection coverage |
| P7-T08 | Spoofing | Model output impersonates trusted authority | TB-03/TB-05 | Human review required; comprehensive authority-claim verification not established |
| P7-T09 | Repudiation | Recommendation accepted without accountable human decision | TB-05/TB-07 | Simulated approval registry and gate; production identity and durable auditability absent |
| P7-T10 | Denial of Service | Repeated inference requests consume local resources | TB-02 | Bounded inference and error handling implemented; rate limiting and load testing incomplete |
| P7-T11 | Spoofing | Caller supplies fabricated approval information | TB-06/TB-07 | Registry-backed approval checks implemented and regression-tested; see P7-F02 |
| P7-T12 | Spoofing | Caller asserts another reviewer's identity | TB-06 | Session-backed approval issuance partially mitigates the tested pathway; no independent authentication; see P7-F03 |
| P7-T13 | Tampering | Model presents untrusted authorization claims as verified facts | TB-02/TB-04 | Evidence-attribution screening implemented and tested; EVT-0007 escalated |
| P7-T14 | Elevation of Privilege | Stale approval survives session revocation or is reused improperly | TB-06/TB-07 | One-time approval consumption implemented; previously issued approval invalidation after session revocation remains unresolved |

Threats are architectural scenarios. Their inclusion does not imply every attack pathway has been demonstrated.

## 6. AI-Specific Attack Scenarios

### P7-A01 — Log-Embedded Prompt Injection

**Attacker capability:** Modify the message field of a synthetic security event.

**Objective:** Influence model conclusions or override trusted triage instructions.

**Observed result:** EVT-0011 caused the model to repeat an unsupported benign-classification claim.

**Implemented mitigation:** Input instruction screening and classification-claim screening detected the tested contamination and required security review.

**Residual risk:** Prompt-level isolation did not prevent contamination. Other payloads may evade pattern-based screening.

### P7-A02 — Evidence Manipulation

**Attacker capability:** Submit misleading event descriptions or fabricated evidence references.

**Objective:** Cause unsupported findings or references.

**Implemented mitigation:** Finding validation checks structure and supporting event identifiers.

**Residual risk:** Valid references do not establish that a conclusion is factually supported.

### P7-A03 — False Escalation

**Attacker capability:** Insert instructions or claims that an event is critical or already approved.

**Objective:** Influence priority or bypass analyst judgment.

**Implemented mitigation:** Structured findings, screening, human review status, and no autonomous response execution.

**Residual risk:** Suggested priorities may still be inaccurate; independent priority ground truth is unavailable.

### P7-A04 — Detection Evasion Through Ambiguous Logs

**Attacker capability:** Construct malicious events with incomplete or misleading descriptions.

**Objective:** Reduce detection and triage effectiveness.

**Observed baseline:** Six true positives, two false positives, one false negative, and three true negatives on the 12-event dataset.

**Residual risk:** Detection coverage is limited to five rules and the synthetic fixture.

### P7-A05 — Resource Exhaustion

**Attacker capability:** Submit malformed, oversized, or repeated event batches.

**Objective:** Degrade processing or inference availability.

**Implemented mitigation:** Input validation, batch limits, bounded inference, and exception recording.

**Residual risk:** Comprehensive load, concurrency, and resource-exhaustion testing has not been demonstrated.

### P7-A06 — Approval Forgery

**Attacker capability:** Supply fabricated approval information to the action gate.

**Objective:** Obtain unauthorized action approval.

**Observed result:** The original approval design accepted caller-provided approval information.

**Implemented mitigation:** Registry-backed approval checks and regression tests.

**Residual risk:** The laboratory approval mechanism is not production-grade authorization.

### P7-A07 — Reviewer Identity Spoofing

**Attacker capability:** Assert a permitted reviewer identity.

**Objective:** Obtain approval without independently authenticated reviewer authority.

**Implemented mitigation:** Simulated reviewer sessions and session-backed approval issuance.

**Residual risk:** Reviewer identity is not authenticated through an external identity provider. Existing approvals may remain valid after session revocation.

### P7-A08 — Unsupported Authorization Attribution

**Attacker capability:** Insert misleading authorization statements into event content.

**Objective:** Cause the model to present authorization as established fact.

**Observed result:** EVT-0007 produced a model finding asserting that a configuration update was approved.

**Implemented mitigation:** Evidence-attribution screening independently flagged the summary and required security review.

**Residual risk:** Pattern matching does not establish actual authorization and may miss alternative wording.

## 7. Security Control Register

| Control ID | Security control | Status |
|---|---|---|
| P7-C01 | Event schema and batch-size validation | Implemented and tested |
| P7-C02 | Deterministic detection baseline | Implemented and tested |
| P7-C03 | Trusted-instruction and untrusted-evidence separation | Prompt-level instructions implemented; effectiveness limited |
| P7-C04 | Supporting event-reference validation | Implemented and tested |
| P7-C05 | Structured AI findings and selected unsupported-claim screening | Implemented and tested; not comprehensive |
| P7-C06 | No autonomous incident-response execution | Implemented and validated in evaluation workflow |
| P7-C07 | Human-review and security-escalation statuses | Implemented and tested; real reviewer authentication absent |
| P7-C08 | Sanitized security and evaluation logging | Synthetic evidence used; production sanitization not validated |
| P7-C09 | Bounded inference and failure handling | Implemented and tested within laboratory scope |
| P7-C10 | Reproducible labeled datasets and evaluation results | Implemented and tested |
| P7-C11 | Input instruction-pattern screening | Implemented and tested |
| P7-C12 | Output classification-claim screening | Implemented and tested |
| P7-C13 | Evidence-attribution screening | Implemented, unit-tested, integration-tested, and live-model validated |
| P7-C14 | Registry-backed approval validation | Implemented and regression-tested |
| P7-C15 | Simulated reviewer sessions and approval issuance | Partially hardened; production authentication absent |
| P7-C16 | Durable approval revocation and independent identity verification | Not implemented |

## 8. Validation Evidence

### 8.1 Deterministic Detection

Evidence: `evidence/baseline_metrics.json`

Observed on the 12-event fixture:

- True positives: 6
- False positives: 2
- False negatives: 1
- True negatives: 3

These results are not production detection benchmarks.

### 8.2 Prompt Injection

Evidence:

- `evidence/prompt_injection_finding.md`
- `evidence/prompt_injection_retest.md`
- `evidence/two_layer_security_validation.md`

EVT-0011 demonstrated model-output contamination and detection by the tested screening controls.

### 8.3 Approval Security

Evidence:

- `evidence/approval_forgery_finding.md`
- `evidence/approval_forgery_retest.md`
- `evidence/approval_issuance_finding.md`
- `evidence/approval_issuance_retest.md`

These artifacts document tested approval-forgery and reviewer-identity weaknesses, mitigations, and remaining limitations.

### 8.4 Evidence Attribution

Evidence:

- `evidence/evidence_attribution_validation.md`
- `evidence/end_to_end_evaluation_three_controls_attributed.json`

The live evaluation completed all 12 synthetic events.

EVT-0007 was flagged exclusively by evidence-attribution screening.

EVT-0011 was flagged by input instruction and classification-claim screening.

Two events required security review. No automated response was permitted.

### 8.5 Regression Tests

The completed evaluation-reporting update passed:

- 11 end-to-end evaluation tests.
- 221 tests across the full regression suite.

These results establish the tested laboratory behavior, not comprehensive security coverage.

## 9. MITRE ATLAS Integration

Project 7 includes a documented MITRE ATLAS mapping for the tested indirect prompt-injection scenario.

**Mapped technique:**
- **AML.T0051 — LLM Prompt Injection**
- **AML.T0051.001 — Indirect**

The mapping relates P7-F01 and P7-A01 to the attacker-controlled instructions embedded in lower-trust security-event content.

The tested controls detected and escalated the selected attack pattern. They did not eliminate the underlying model-output contamination.

The complete mapping, including framework references, attack prerequisites, implemented controls, validation evidence, and residual risks, is documented in the [Security Framework Mapping](security-framework-mapping.md).

Additional ATLAS techniques are not assigned to P7-F02 or P7-F03 without evidence supporting a specific technique-level relationship.

Framework alignment does not establish complete mitigation, production security assurance, or compliance.

## 10. Residual Risks

### Observed or Demonstrated

- Prompt injection contaminated EVT-0011 model output.
- Unsupported authorization attribution occurred in EVT-0007.
- Earlier approval handling accepted fabricated caller-provided approval information.
- Reviewer identity assurance remains insufficient for production authorization.

### Potential or Incompletely Evaluated

- Novel prompt-injection patterns may evade deterministic screening.
- Unusual wording may bypass evidence-attribution detection.
- Validated event references may support misleading conclusions.
- Model-generated priorities may be inaccurate.
- Previously issued approvals may survive session revocation.
- In-memory approval records do not provide durable auditability.
- Local inference may be vulnerable to resource exhaustion.
- Real log sources may require authentication and provenance verification.
- Production environments require stronger privacy and access controls.

## 11. Implementation Status

**Implemented and tested:** Event validation, deterministic detection, structured model findings, selected input and output screening, evidence-attribution screening, evaluation reporting, and registry-backed approval-gate restrictions.

**Partially implemented:** Reviewer-session assurance, approval issuance, authorization lifecycle protections, and semantic evidence assessment.

**Not implemented or not established:** Production identity verification, durable approval revocation, authenticated event provenance, comprehensive claim verification, production response integration, and operational effectiveness benchmarks.

**Validation status:** 221 regression tests passed. The documented three-control live evaluation completed 12 of 12 synthetic events, with two security-review escalations and no automated response authorization.

**Final status:** Functional security architecture laboratory with demonstrated findings, tested mitigations, and explicitly documented residual risks. Not production-ready.