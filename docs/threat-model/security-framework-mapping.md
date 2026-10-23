# Project 7 — Security Framework Mapping

**Project:** AI-Assisted Security Operations
**Mapping date:** 2026-10-24
**Scope:** Synthetic AI-assisted security-event triage and simulated approval authorization
**Frameworks:** NIST AI RMF 1.0, NIST CSF 2.0, OWASP LLM Top 10 (2026), MITRE ATLAS

## 1. Purpose and Scope

This document maps Project 7's security architecture, identified threats, implemented controls, validation evidence, and residual risks to recognized security frameworks.

The mappings demonstrate architectural relevance and traceability. They do not establish certification, regulatory compliance, production readiness, or complete mitigation of the mapped risks.

Project 7 processes synthetic security events through deterministic detection, local language-model inference, structured finding validation, three security-screening controls, and a separate simulated authorization workflow.

No operational incident-response actions are executed.

### Evidence Classification

The following status definitions apply throughout this document:

- **Implemented and tested:** Code exists and defined laboratory tests have passed.
- **Partially mitigated:** Tested attack paths were blocked or escalated, but material residual risks remain.
- **Proposed:** Architectural mitigation documented but not implemented or validated.
- **Not established:** Available evidence is insufficient to demonstrate the claimed security property.

A framework mapping does not change a control's implementation status.

## 2. Framework Versions and Sources

| Framework | Reference edition | Mapping approach |
|---|---|---|
| NIST AI RMF | AI RMF 1.0 | Risk-management functions and documented evidence |
| NIST CSF | CSF 2.0 | Cybersecurity governance, protection, detection, and response boundaries |
| OWASP LLM Top 10 | 2026 | LLM application risks relevant to tested architecture |
| MITRE ATLAS | Current catalog, reviewed October 2026 | Adversarial AI techniques supported by observed test behavior |

Official sources:

- [NIST AI RMF 1.0](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf)
- [NIST AI RMF Playbook](https://airc.nist.gov/airmf-resources/)
- [NIST CSF 2.0](https://www.nist.gov/cyberframework)
- [OWASP LLM Top 10 — 2026](https://github.com/GenAI-Security-Project/GenAI-LLM-Top10/tree/main/2026/final)
- [MITRE ATLAS](https://atlas.mitre.org/)

Framework descriptions are paraphrased for architectural mapping; they are not reproduced as normative control requirements.

## 3. NIST AI RMF Mapping

The NIST AI RMF organizes AI risk management into four functions: GOVERN, MAP, MEASURE, and MANAGE.

| AI RMF function | Project 7 application | Implementation and evidence | Limitation |
|---|---|---|---|
| GOVERN | Define intended use, decision authority, review requirements, and risk ownership | Documented architecture, trust policies, threat model, and residual risks | No operational governance organization or production oversight process |
| MAP | Identify untrusted inputs, AI trust boundaries, authorization threats, and affected assets | STRIDE threat model, seven trust boundaries, P7-F01 through P7-F03 | Synthetic scope; no production environment or asset inventory |
| MEASURE | Evaluate deterministic detection, model behavior, screening effectiveness, and security regressions | Baseline metrics, 12-event live evaluation, 221-test regression suite | No representative production dataset or validated priority-accuracy benchmark |
| MANAGE | Apply screening, require human review, separate recommendations from simulated authorization, and track residual risks | Three screening controls, review statuses, approval-gate tests | Human authentication and production response enforcement are not established |

### 3.1 GOVERN — Risk Governance

**Architectural alignment**

The project explicitly defines AI-assisted triage as decision support rather than an autonomous response authority.

The architecture documents control ownership by component, review requirements, trust boundaries, and unresolved security limitations.

**Evidence**

- [System Architecture](../architecture/system-architecture.md)
- [Evidence Trust Policy](../architecture/evidence-trust-policy.md)
- [STRIDE Threat Model](stride-threat-model.md)

**Status:** Documented in the laboratory; organizational governance not implemented.

### 3.2 MAP — Risk Identification

**Architectural alignment**

The threat model identifies risks involving:

- Untrusted event content.
- Prompt-injection contamination.
- Unsupported model-generated assertions.
- Approval forgery.
- Reviewer identity spoofing.
- Weak approval provenance and revocation.

**Evidence**

- [STRIDE Threat Model](stride-threat-model.md)
- [Prompt Injection Finding](../../evidence/prompt_injection_finding.md)
- [Approval Forgery Finding](../../evidence/approval_forgery_finding.md)
- [Approval Issuance Finding](../../evidence/approval_issuance_finding.md)

**Status:** Implemented as documented laboratory threat analysis.

### 3.3 MEASURE — Evaluation and Validation

**Architectural alignment**

The project uses reproducible tests and archived evaluation evidence to assess defined security behaviors.

The deterministic detection baseline on 12 synthetic events recorded:

- True positives: 6
- False positives: 2
- False negatives: 1
- True negatives: 3
- Precision: 0.75
- Recall: approximately 0.857
- F1 score: 0.80
- Accuracy: 0.75

The latest recorded live-model evaluation completed all 12 events, with two security-review escalations and no automated responses permitted.

The latest complete regression suite passed 221 tests.

**Evidence**

- [End-to-End Evaluation Report](../../evidence/end_to_end_evaluation_report.md)
- [Evidence Attribution Validation](../../evidence/evidence_attribution_validation.md)

**Status:** Implemented and tested on the defined synthetic scenarios.

**Limitation:** These results do not establish operational detection effectiveness, analyst productivity gains, or model-priority accuracy.

### 3.4 MANAGE — Risk Treatment

**Architectural alignment**

The project implements screening and escalation controls, restricts simulated authorization, and documents residual risks.

**Implemented and tested**

- Input instruction screening.
- Output classification-claim screening.
- Evidence-attribution screening.
- Mandatory human-review status for completed findings.
- Registry-backed simulated approval checks.

**Partially mitigated**

- Prompt-injection contamination.
- Reviewer identity and approval issuance.

**Proposed**

- Independent human authentication.
- Protected approval issuance.
- Durable audit records.
- Comprehensive approval revocation.
- Production response authorization and execution controls.

**Status:** Partial risk treatment demonstrated; production risk management not established.

## 4. NIST CSF 2.0 Mapping

The following mappings are at the function and category level. They describe architectural relevance rather than verified conformity with individual CSF outcomes.

| CSF function / category | Project 7 relevance | Evidence | Status |
|---|---|---|---|
| GV.RM — Risk Management Strategy | Defined synthetic scope, security findings, residual-risk treatment | Threat model and findings | Documented; operational strategy not established |
| ID.RA — Risk Assessment | Threat identification, attack reproduction, and residual-risk analysis | STRIDE model and P7 findings | Implemented as laboratory analysis |
| PR.AA — Identity Management, Authentication, and Access Control | Simulated reviewer sessions and registry-backed approval authorization | P7-F02 and P7-F03 retests | Partially mitigated; genuine authentication absent |
| DE.CM — Continuous Monitoring | Security-event processing and detection concepts | Rule engine and evaluation report | Laboratory detection demonstrated; continuous operational monitoring absent |
| DE.AE — Adverse Event Analysis | Detection rules, AI-assisted triage, evidence screening, and escalation | Evaluation and screening evidence | Implemented and tested on synthetic events |
| RS.MA — Incident Management | Human-review requirement and simulated response-approval decisions | Approval-gate evidence | Simulated only; no operational incident management |

### Identity and Authorization Qualification

The PR.AA mapping is especially important.

Project 7 demonstrates selected access-control behaviors, including approval-token binding, expiration, single-use consumption, and rejection of the tested forged-approval path.

It does not demonstrate independent authentication of human reviewers.

A simulated session is not equivalent to an authenticated enterprise identity.

Therefore, the PR.AA mapping must remain **partial**, with production identity assurance explicitly unresolved.

## 5. OWASP LLM Top 10 — 2026 Mapping

### 5.1 LLM01:2026 — Prompt Injection

**Related findings:** P7-F01
**Related attack:** P7-A01
**Test event:** EVT-0011

**Threat**

Attacker-controlled instructions embedded in security-event content attempt to influence the model's classification of a malicious event.

The tested payload instructed the model to ignore prior instructions and mark the event benign.

**Observed behavior**

The model produced an unsupported benign-classification assertion.

**Implemented controls**

- Input instruction screening.
- Output classification-claim screening.
- Structured finding validation.
- Security-review escalation.

**Validation**

Both screening layers detected the tested contamination and escalated the finding to `security_review_required`.

The model-generated assertion remained contaminated.

**Evidence**

- [Prompt Injection Finding](../../evidence/prompt_injection_finding.md)
- [Prompt Injection Retest](../../evidence/prompt_injection_retest.md)
- [Two-Layer Security Validation](../../evidence/two_layer_security_validation.md)

**Assessment:** Partially mitigated.

**Residual risk:** Novel prompt-injection phrasing may evade pattern-based screening, and model-output contamination remains possible.

### 5.2 LLM03:2026 — Excessive Agency

**Related findings:** P7-F02 and P7-F03
**Related architecture:** Simulated approval authorization boundary

**Threat**

If AI-generated recommendations or untrusted approval assertions were allowed to trigger privileged response actions, the system could exceed its intended decision-support authority.

**Implemented controls**

- Separation of model-generated findings from approval authorization.
- Rejection of caller-supplied approval dictionaries.
- Registry-backed approval-token validation.
- Finding/action binding.
- Approval expiration and single-use consumption.
- No operational response executor.

**Validation**

The tested approval-dictionary forgery was rejected.

Simulated reviewer-session checks rejected direct reviewer-identifier impersonation and identity mismatches.

No response action was executed.

**Evidence**

- [Approval Forgery Finding](../../evidence/approval_forgery_finding.md)
- [Approval Forgery Retest](../../evidence/approval_forgery_retest.md)
- [Approval Issuance Finding](../../evidence/approval_issuance_finding.md)
- [Approval Issuance Retest](../../evidence/approval_issuance_retest.md)

**Assessment:** Relevant architectural risk with partial laboratory controls.

**Residual risk:** Reviewer-session establishment remains unauthenticated, and protected approval issuance is not implemented.

The project does not demonstrate exploitation of an autonomous agent with operational tool permissions.

### 5.3 LLM07:2026 — Misinformation

**Related evidence:** EVT-0007 and EVT-0011

**Threat**

The language model may generate incorrect or unsupported security conclusions that appear credible to an analyst.

**Observed behaviors**

- EVT-0007: Unsupported assertion that a configuration update was approved.
- EVT-0011: Unsupported benign-classification assertion.

**Implemented controls**

- Classification-claim screening.
- Evidence-attribution screening.
- Review-status escalation.
- Explicit separation of event facts from model interpretation.

**Validation**

The latest three-control evaluation escalated both events for security review.

**Evidence**

- [Evidence Attribution Validation](../../evidence/evidence_attribution_validation.md)
- [End-to-End Evaluation Report](../../evidence/end_to_end_evaluation_report.md)

**Assessment:** Selected unsupported claims detected and escalated.

**Residual risk:** Comprehensive factual verification and independent evidence corroboration are not implemented.

### 5.4 LLM10:2026 — Improper Output Handling

**Related architecture:** Structured finding validation and downstream screening

**Threat**

A downstream application may treat model-generated content as trustworthy without sufficient validation.

**Implemented controls**

- Structured output validation.
- Supporting-event reference checks.
- Classification-claim screening.
- Evidence-attribution screening.
- Separation between findings and simulated action authorization.

**Evidence**

- [System Architecture](../architecture/system-architecture.md)
- [Evidence Trust Policy](../architecture/evidence-trust-policy.md)
- [End-to-End Evaluation Report](../../evidence/end_to_end_evaluation_report.md)

**Assessment:** Selected output validation and screening controls implemented and tested.

**Residual risk:** Schema validity does not establish factual accuracy. Investigation recommendations are not comprehensively checked for unsafe or unsupported instructions.

No model-output-driven code execution or production action execution was demonstrated.

## 6. MITRE ATLAS Mapping

MITRE ATLAS describes adversarial techniques targeting AI-enabled systems.

The following technique is mapped because its mechanism is supported by the documented Project 7 attack scenario.

### AML.T0051 — LLM Prompt Injection

**Relevant sub-technique:** AML.T0051.001 — Indirect

**Related finding:** P7-F01
**Related attack:** P7-A01
**Test event:** EVT-0011

**Attack mechanism**

The adversarial instruction is embedded in a lower-trust security-event message that the language model processes during an otherwise legitimate triage task.

The attacker does not need to control the system's trusted instructions.

The injected event content attempts to redirect the model toward an unsupported benign classification.

**Observed result**

The model repeated an unsupported benign assertion.

**Implemented and tested mitigations**

- P7 input instruction screening.
- P7 output classification-claim screening.
- Security-review escalation.
- Structured finding validation.

**Validation evidence**

- [Prompt Injection Finding](../../evidence/prompt_injection_finding.md)
- [Prompt Injection Retest](../../evidence/prompt_injection_retest.md)
- [Two-Layer Security Validation](../../evidence/two_layer_security_validation.md)

**Residual risk**

The underlying model behavior was not corrected.

The implemented screening controls recognize selected patterns and cannot guarantee detection of all indirect prompt injections.

**Assessment:** Technique reproduced in a synthetic laboratory; tested detection and escalation controls partially mitigate downstream risk.

### ATLAS Mapping Boundaries

P7-F02 and P7-F03 are application authorization and identity-trust vulnerabilities.

They are documented through STRIDE analysis and mapped to relevant NIST CSF and OWASP architectural concerns.

They are not assigned additional MITRE ATLAS technique identifiers without a supported technique-level match.

Likewise, EVT-0007 is evidence of an unsupported model-generated authorization assertion. It is not independently established as a separate adversarial AI technique.

This conservative approach avoids inventing or overextending technique mappings.

## 7. Cross-Framework Traceability Matrix

| Project 7 risk or finding | Framework mapping | Implemented control or mitigation | Validation evidence | Residual risk |
|---|---|---|---|---|
| P7-F01 — Prompt injection | NIST AI RMF MAP / MEASURE / MANAGE; OWASP LLM01:2026; ATLAS AML.T0051.001 | Input and classification-claim screening | Prompt-injection retest; two-layer validation | Model contamination and screening bypass |
| P7-F02 — Approval forgery | NIST CSF PR.AA; OWASP LLM03:2026 architectural alignment | Registry-backed authorization and token validation | Approval-forgery retest | Approval issuance and reviewer identity assurance |
| P7-F03 — Reviewer impersonation | NIST CSF PR.AA; NIST AI RMF MANAGE | Simulated reviewer sessions and identity matching | Approval-issuance retest | Unauthenticated session establishment |
| EVT-0007 — Unsupported approval assertion | NIST AI RMF MEASURE / MANAGE; OWASP LLM07:2026 | Evidence-attribution screening | Evidence-attribution validation; end-to-end evaluation | Incomplete claim verification |
| EVT-0011 — Unsupported benign classification | NIST AI RMF MEASURE / MANAGE; OWASP LLM01:2026 and LLM07:2026 | Input and output classification screening | Prompt-injection and evaluation evidence | Continued model-output contamination |
| Unsafe downstream handling of model findings | NIST CSF DE.AE; OWASP LLM10:2026 | Structured validation and review escalation | Regression tests; evaluation report | Unsupported recommendations may remain |

The matrix uses Project 7 finding identifiers where supported. It does not claim that every STRIDE threat has a distinct ATLAS technique or OWASP category.

## 8. Implemented, Partial, and Proposed Controls

### Implemented and Tested

- Synthetic event validation.
- Five deterministic detection rules.
- Structured model-finding validation.
- Input instruction screening.
- Output classification-claim screening.
- Evidence-attribution screening.
- Security-review escalation.
- Registry-backed simulated approval checks.
- Approval expiration and single-use consumption.
- Simulated reviewer-session validation and identity matching.
- Synthetic evaluation and automated regression testing.

### Partially Mitigated

- P7-F01: Tested prompt-injection contamination detected and escalated; model output remains vulnerable.
- P7-F02: Original approval-dictionary forgery rejected; genuine human authorization not established.
- P7-F03: Direct reviewer-identifier impersonation rejected; session establishment remains unauthenticated.
- Evidence attribution: Selected unsupported positive claims detected; comprehensive claim verification absent.

### Proposed or Not Established

- Authenticated event provenance.
- Independent human identity verification.
- Protected reviewer-session establishment.
- Production reviewer role enforcement.
- Durable, tamper-resistant approval audit records.
- Comprehensive approval revocation.
- Transactional approval consumption across processes.
- Independent corroboration of model-generated claims.
- Comprehensive semantic prompt-injection detection.
- Production incident-response execution.
- Operational effectiveness and analyst-productivity benchmarks.

## 9. Validation Summary

The latest archived three-control evaluation recorded:

| Metric | Result |
|---|---|
| Synthetic events attempted | 12 |
| Events completed | 12 |
| Inference failures | 0 |
| Security-review escalations | 2 |
| Automated responses permitted | 0 |
| Latest complete regression suite | 221 passing tests |

The two security-review escalations involved:

- **EVT-0007:** Evidence-attribution screening detected an unsupported approval assertion.
- **EVT-0011:** Input instruction screening and classification-claim screening detected the tested prompt-injection contamination.

These observations validate the defined synthetic test cases, not general production security effectiveness.

## 10. Architectural Conclusions

Project 7 demonstrates how an AI-assisted security triage system can combine deterministic detection, local LLM inference, structured output validation, independent security screening, and a separate simulated approval boundary.

The framework mappings establish traceability between observed attack scenarios, architectural controls, validation evidence, and unresolved risks.

The most significant demonstrated findings are:

1. Untrusted log content can contaminate model-generated security conclusions.
2. Pattern-based screening can detect selected contamination without correcting the underlying model behavior.
3. Registry-backed authorization can reject caller-fabricated approval assertions.
4. Approval-token integrity does not establish trustworthy human approval issuance.
5. Simulated reviewer sessions do not provide independent human authentication.
6. AI-generated security findings must remain separate from privileged response authority.

**Final assessment:** Project 7 demonstrates selected security controls and risk-management practices aligned with NIST, OWASP, and MITRE ATLAS. The evidence supports laboratory-level validation only. Production-grade identity assurance, evidence verification, approval governance, and response enforcement remain outside the implemented scope.