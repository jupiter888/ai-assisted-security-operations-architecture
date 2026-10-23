# Project 7 — AI-Assisted Security Operations

## 1. Project Objective

Design, implement, and evaluate a controlled AI-assisted security-event triage workflow using synthetic security logs.

The project examines deterministic detection, language-model-assisted investigation, evidence attribution, prompt-injection risks, and human authorization boundaries.

The system is an educational security architecture laboratory. It does not execute autonomous incident-response actions or establish production security guarantees.

## 2. System Architecture

The architecture consists of seven logical components.

### 2.1 Synthetic Security Event Dataset

The dataset contains 12 synthetic events with separate malicious/benign ground-truth classifications.

Events use fictional identities, resources, and documentation-range IP addresses.

The dataset supports reproducible testing but is too small to establish production detection performance.

### 2.2 Event Validation and Normalization

`src/security/event_validator.py` validates event structure, required fields, accepted data types, timestamps, and batch-size limits.

Validation rejects malformed records but does not authenticate event provenance or establish the truth of message content.

### 2.3 Deterministic Detection Engine

`src/detection/rule_engine.py` implements five deterministic detection rules, P7-R01 through P7-R05.

The engine produces reproducible alerts and serves as the detection baseline.

The 12-event baseline produced:

- True positives: 6
- False positives: 2
- False negatives: 1
- True negatives: 3
- Precision: 0.75
- Recall: approximately 0.857
- F1 score: 0.80
- Accuracy: 0.75

These measurements apply only to the synthetic evaluation dataset.

### 2.4 AI-Assisted Triage Engine

`src/triage/ollama_client.py` interfaces with a locally hosted Ollama model, `qwen2.5:3b`.

`src/triage/triage_pipeline.py` constructs the event-analysis prompt and coordinates validation, model inference, and security screening.

The model generates a structured finding containing a summary, suggested priority, supporting event identifiers, investigation recommendations, and uncertainty notes.

`src/security/finding_validator.py` validates the finding's expected structure and event references.

Schema-valid findings may still contain unsupported or misleading assertions.

### 2.5 Security Screening and Enforcement

The workflow implements three distinct screening controls:

**Control 1 — Input Instruction Screening**

`src/security/log_content_screen.py` detects selected instruction-like patterns embedded in event content.

**Control 2 — Output Classification-Claim Screening**

`src/security/claim_consistency.py` detects selected unsupported classification claims in model-generated findings.

**Control 3 — Evidence-Attribution Screening**

`src/security/evidence_attribution.py` detects selected unsupported positive approval, authorization, or verification assertions in model-generated findings.

The controls are deterministic and pattern-based. They do not provide complete semantic understanding or comprehensive prompt-injection prevention.

A positive result from any control changes the finding's status to `security_review_required`.

Otherwise, a successfully processed finding receives `pending_human_review`.

Neither status constitutes permission to execute a response action.

The triage pipeline sets `automated_response_permitted` to `False`.

### 2.6 Evaluation Framework

`src/evaluation/end_to_end_evaluation.py` evaluates the synthetic events individually.

The framework records:

- Deterministic detection results
- Model-generated priorities and supporting event references
- Validated model findings
- Input instruction screening results
- Output classification-claim screening results
- Evidence-attribution screening results
- Review status and inference failures
- Whether automated response was permitted

The ground truth distinguishes malicious from benign events. It does not contain independently established incident-priority labels.

Consequently, observed model priorities are recorded but priority accuracy is not established.

The evaluation does not measure analyst productivity or demonstrate that AI-assisted triage improves detection performance.

### 2.7 Human Review and Approval Boundary

Findings are intended for human analyst review.

The project also contains a simulated reviewer-session and approval workflow using:

- `src/security/reviewer_sessions.py`
- `src/security/approval_registry.py`
- `src/security/approval_gate.py`

The approval gate was hardened against caller-supplied fabricated approval dictionaries.

The simulated session and registry mechanisms are not production identity and access management systems.

Reviewer identity is not independently authenticated through a production identity provider.

Approval mechanisms must not be interpreted as proof that a real human authorized an operational action.

No incident-response action is executed by the evaluation workflow.

## 3. Trust Boundaries

### TB-01 — Event Source to Event Validation

Synthetic log records enter as untrusted data.

Schema validation establishes structural acceptability, not authenticity, authorization, or factual correctness.

### TB-02 — Validated Events to Language Model

Validated event fields and messages are passed into the model's context.

Event messages may contain attacker-controlled instructions or unsupported assertions.

Prompt instructions explicitly describe these records as untrusted evidence.

These instructions reduce ambiguity about the intended trust boundary but do not guarantee model compliance.

### TB-03 — Language Model to Finding Validation

Model-generated JSON crosses from probabilistic inference into structured application processing.

Finding validation checks schema and supporting event references.

It does not prove that the model's conclusions follow from the supplied evidence.

### TB-04 — Model Findings to Security Screening

Validated findings undergo classification-claim and evidence-attribution screening.

The screening controls identify selected suspicious assertions and require additional review.

A negative screening result is not evidence that the finding is correct or trustworthy.

### TB-05 — Triage Output to Human Analyst

Findings, priorities, uncertainty notes, and security-review statuses are presented as decision-support information.

Human reviewers must distinguish observed event facts from unverified model conclusions.

### TB-06 — Reviewer Identity to Approval Registry

The simulated reviewer-session mechanism provides an in-memory approval workflow.

It does not establish production-grade authentication, durable auditability, or independently verified human identity.

### TB-07 — Approval Registry to Action Authorization

The approval gate checks permitted action types, approval records, review state, and applicable restrictions.

The laboratory does not establish an end-to-end production authorization system.

No autonomous incident-response execution is implemented in the evaluated workflow.

## 4. Security Findings and Validation Evidence

### 4.1 P7-F01 — Prompt-Injection Contamination

EVT-0011 contains instruction-like content intended to influence model classification.

The model repeated an unsupported benign-classification claim.

Input instruction screening and output classification-claim screening detected the tested contamination and required security review.

The model output was not automatically corrected.

### 4.2 P7-F02 — Approval Forgery

An earlier approval-gate implementation accepted caller-provided approval information.

The gate was hardened to require registry-backed approval checks.

This addresses the tested forgery pathway but does not establish production-grade authorization.

### 4.3 P7-F03 — Reviewer Identity and Approval Issuance

The laboratory evaluated weaknesses associated with caller-asserted reviewer identity and approval issuance.

Session-backed approval controls provide a partial mitigation.

Independent human identity verification remains unresolved. Revoking a simulated reviewer session does not invalidate approval tokens issued before revocation; those approvals remain valid until consumed or expired. Durable approval records and comprehensive revocation enforcement are not implemented.

### 4.4 Unsupported Authorization Assertion — EVT-0007

The model described a configuration update as approved without independently verified authorization evidence.

Evidence-attribution screening flagged the summary and independently escalated the finding to security review.

This is a demonstrated model-output attribution risk, not proof that the underlying update was unauthorized.

## 5. Live Evaluation Results

The archived three-control evaluation is stored at:

`evidence/end_to_end_evaluation_three_controls_attributed.json`

Observed results:

| Metric | Result |
|---|---|
| Events attempted | 12 |
| Events completed | 12 |
| Inference failures | 0 |
| Security-review escalations | 2 |
| Unsafe authorization flags | 0 |
| Automated responses permitted | 0 |

EVT-0007 was flagged by evidence-attribution screening.

EVT-0011 was flagged by input instruction screening and output classification-claim screening.

All remaining events received `pending_human_review` in the recorded evaluation.

The full regression suite passed 221 tests following the evaluation-reporting changes.

These results validate the documented behavior on the tested cases, not comprehensive security effectiveness.

## 6. Residual Risks

The architecture retains several limitations:

- Prompt injection may contaminate model output despite explicit trust-boundary instructions.
- Pattern-based screening can miss novel wording, paraphrases, and complex multi-clause assertions.
- Schema validation does not authenticate event provenance.
- Model-generated priorities have not been validated against independent priority labels.
- Investigation recommendations are not comprehensively screened for unsafe or unsupported content.
- Reviewer sessions do not provide production-grade identity assurance.
- In-memory approvals lack durable, independently authenticated authorization evidence.
- Approval revocation and session lifecycle controls require further hardening.
- Synthetic evaluation results do not generalize to operational security environments.
- Human review remains necessary for every generated finding.

## 7. Security Design Principles

The project follows these principles:

1. Treat security-event content as untrusted evidence.
2. Separate deterministic detection from model-generated interpretation.
3. Validate structured model output before downstream use.
4. Detect selected instruction and evidence-contamination patterns.
5. Escalate suspicious findings without automatically treating them as verified.
6. Keep model-generated recommendations separate from action authorization.
7. Preserve security findings, test results, and residual risks as auditable evidence.
8. Distinguish proposed controls from implemented and tested controls.

## 8. Implementation and Validation Status

**Implemented and tested:** Synthetic event generation, event validation, deterministic detection, local AI-assisted triage, structured finding validation, three screening controls, evaluation reporting, and simulated approval-gate restrictions.

**Partially implemented or limited:** Reviewer-session assurance, approval issuance and revocation controls, evidence attribution, and prompt-injection detection.

**Not established:** Production identity verification, authenticated event provenance, comprehensive semantic claim verification, production incident-response integration, and operational detection effectiveness.

**Architecture status:** Functional synthetic security laboratory with documented security tests and residual risks. Not production-ready.