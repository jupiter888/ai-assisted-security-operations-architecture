# Project 7 — Synthetic Security Event Schema

## 1. Purpose

Define the canonical event format used to evaluate deterministic security detection and AI-assisted incident triage.

Both approaches operate on equivalent synthetic event evidence.

Ground-truth labels are maintained separately and must not be exposed to detection or triage components during inference.

**Status:** Implemented and tested within the synthetic laboratory.

## 2. Canonical Event Structure

Each event is represented as a JSON object containing exactly ten required fields.

| Field | Type | Required | Description |
|---|---|---|---|
| event_id | string | Yes | Synthetic event identifier |
| timestamp | string | Yes | UTC timestamp with timezone information |
| event_type | string | Yes | Supported security-event category |
| source | string | Yes | Synthetic event-source identifier |
| actor_id | string | Yes | Synthetic user or service identifier |
| source_ip | string | Yes | IPv4 address from a reserved documentation network |
| target | string | Yes | Synthetic resource identifier |
| action | string | Yes | Observed action |
| outcome | string | Yes | success, failure, or blocked |
| message | string | Yes | Untrusted descriptive event content |

Unknown or missing fields are rejected.

All values must be nonempty strings of no more than 1,024 characters.

## 3. Example Synthetic Event

```json
{
  "event_id": "EVT-0001",
  "timestamp": "2026-10-01T10:15:00Z",
  "event_type": "authentication",
  "source": "synthetic-identity-service",
  "actor_id": "user-001",
  "source_ip": "192.0.2.10",
  "target": "synthetic-application",
  "action": "login",
  "outcome": "failure",
  "message": "Authentication attempt rejected."
}
```

All identifiers and resource names are synthetic.

The example illustrates the schema and is not a representation of the actual EVT-0001 dataset record.

## 4. Supported Event Categories

The validator permits exactly six categories:

- authentication
- authorization
- network
- process_execution
- configuration_change
- data_access

Unsupported categories are rejected.

The permitted outcomes are:

- success
- failure
- blocked

## 5. Timestamp Validation

The validator uses Python's `datetime.fromisoformat()` after converting a trailing `Z` into `+00:00`.

The timestamp must include timezone information and represent a UTC offset of zero.

For example:

- `2026-10-01T10:15:00Z`
- `2026-10-01T10:15:00+00:00`

The validator does not enforce one canonical timestamp serialization.

It also does not normalize accepted timestamps into a common output format.

## 6. Source IP Validation

The validator accepts IPv4 addresses only.

Addresses must belong to one of these documentation networks:

| Network | Purpose |
|---|---|
| `192.0.2.0/24` | TEST-NET-1 |
| `198.51.100.0/24` | TEST-NET-2 |
| `203.0.113.0/24` | TEST-NET-3 |

IPv6 addresses and IPv4 addresses outside these networks are rejected.

Passing this check does not authenticate the source of an event.

## 7. Batch Validation

**Implementation:** `src/security/event_validator.py`

The validator enforces:

- Input batch must be a list.
- Maximum batch size of 1,000 events.
- Every event must match the exact required field set.
- Every field must contain a nonempty string of at most 1,024 characters.
- Event categories and outcomes must be permitted.
- Timestamps must parse with a UTC offset.
- Source IPs must belong to approved documentation networks.
- Event identifiers must be unique within the batch.

The validator returns copies of accepted event dictionaries.

It does not perform semantic normalization, authenticate event provenance, or guarantee factual accuracy.

An empty list is permitted by the current batch validator.

## 8. Ground-Truth Labels

Ground truth is stored separately from canonical event records.

The synthetic evaluation dataset includes labels describing intended scenario classifications.

| Field | Type | Description |
|---|---|---|
| event_id | string | References a synthetic event |
| classification | string | benign or malicious |
| scenario_id | string | Synthetic scenario identifier |
| expected_detection | boolean | Whether a baseline detection is expected |
| rationale | string | Explanation of the synthetic label |

Ground-truth information is reserved for evaluation.

Detection and AI triage must not receive ground-truth classifications, expected detection results, or evaluation rationales as inference inputs.

The current dataset contains twelve synthetic events: seven malicious and five benign.

These classifications are synthetic evaluation labels, not independently verified real-world incident outcomes.

## 9. Detection and Triage Interfaces

### Deterministic Detection

**Implementation:** `src/detection/rule_engine.py`

The deterministic engine evaluates validated event evidence using five rules:

- P7-R01
- P7-R02
- P7-R03
- P7-R04
- P7-R05

The baseline evaluation produced:

| Metric | Result |
|---|---|
| True positives | 6 |
| False positives | 2 |
| False negatives | 1 |
| True negatives | 3 |
| Precision | 0.75 |
| Recall | Approximately 0.8571 |
| F1 | 0.80 |
| Accuracy | 0.75 |

These metrics apply only to the twelve-event synthetic fixture.

### AI-Assisted Triage

**Implementation:** `src/triage/triage_pipeline.py`

The workflow processes validated events and obtains structured findings from a local language model.

Finding fields include:

- finding_id
- summary
- suggested_priority
- supporting_event_ids
- investigation_recommendations
- uncertainty_notes

The finding validator checks structure and supporting event references.

The pipeline also applies three separate security-screening controls:

1. Input instruction screening.
2. Output classification-claim screening.
3. Evidence-attribution screening.

These controls do not comprehensively verify factual accuracy.

All completed findings remain subject to human review.

## 10. Dataset Design and Coverage

**Generator:** `src/evaluation/generate_dataset.py`

**Event data:** `data/synthetic/events.json`

**Ground truth:** `data/synthetic/ground_truth.json`

The synthetic scenarios cover:

- Normal authentication activity.
- Authentication failures.
- Authorization attempts.
- Data access and export.
- Configuration changes.
- Network activity.
- Process execution.
- Benign events resembling suspicious activity.
- Prompt injection embedded in an event message.
- Ambiguous event evidence.

The dataset supports reproducible laboratory testing.

Its size and construction do not support claims about operational detection effectiveness.

## 11. Security and Privacy Constraints

- Use synthetic identities and resource names.
- Restrict source IPs to documentation networks.
- Do not include real credentials, tokens, or private keys.
- Do not include customer data or confidential infrastructure details.
- Treat all event-message content as untrusted.
- Keep evaluation ground truth separate from analysis inputs.
- Review generated reports before publication.
- Do not treat schema validation as authentication or authorization.
- Do not permit model-generated findings to execute response actions.

## 12. Validation Evidence

The implementation is covered by the project's regression tests.

The last completed full regression run passed 221 tests.

The archived three-control live evaluation processed all twelve synthetic events successfully.

**Evaluation artifact:**

`evidence/end_to_end_evaluation_three_controls_attributed.json`

Two events required security review:

- EVT-0007: Unsupported approval attribution.
- EVT-0011: Prompt-injection contamination.

No automated response was permitted in the evaluated workflow.

## 13. Implementation Status

**Implemented and tested:**

- Canonical ten-field event schema.
- Event category and outcome restrictions.
- UTC timestamp validation.
- Documentation-range IPv4 validation.
- Maximum field lengths.
- Maximum batch size.
- Duplicate event-ID rejection.
- Synthetic dataset generation.
- Separate evaluation ground truth.
- Deterministic detection and AI-assisted triage workflows.

**Not established:**

- Authenticity of event sources.
- Cryptographic event integrity.
- Independent verification of telemetry accuracy.
- Semantic consistency of all event fields.
- Production ingestion security.
- Operational detection effectiveness.

**Final status:** Implemented and tested synthetic event schema with documented trust limitations. Not production-ready.