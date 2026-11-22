# HSOC — Detection & Response Specification

**Version:** 1.1-draft | **Mode:** conservative hybrid response

## 1. Pipeline

```text
OS source
  -> Collector
  -> Validation
  -> Normalization
  -> Event persistence
  -> Rules
  -> Detection
  -> Dedup/Correlation
  -> Incident
  -> Dashboard/Notification
  -> Proposed action (simulation/approval)
```

Every step must have error handling and structured logging.

## 2. Canonical event

Logical fields:
- `event_id`
- `source`
- `event_type`
- `occurred_at`
- `received_at`
- local `host_id`
- optional `severity_hint`
- validated `attributes`
- `raw_reference` or redacted payload
- `schema_version`

Raw data is untrusted. Limit size and retained fields.

## 3. Rules

Every rule includes:
- stable ID;
- version;
- description;
- category;
- conditions;
- severity;
- confidence;
- required sources;
- explanation of the match;
- any exceptions;
- positive and negative tests;
- enabled/disabled state.

Rules must be deterministic for the same input and version. Avoid opaque rules without justification.

## 4. Severity and confidence

Severity measures potential impact; confidence measures how much the signal supports the conclusion. Do not confuse the two values. Show both and explain how they were assigned.

## 5. False positives and negatives

- Every rule must have positive and negative fixtures.
- The user can mark a detection as a false positive with a reason.
- A rule correction requires a new version and regression.
- Do not globally suppress similar events without analysis.
- Monitor precision/recall per category.
- Do not promise zero false positives.

## 6. Dedup and correlation

- Dedup on fingerprint and a documented window.
- Keep count and first/last occurrence.
- Correlate only events with an explicit, testable relationship.
- Do not merge incidents based on vague similarity.
- Keep event-detection-incident links.

## 7. Incident lifecycle

States: `OPEN`, `ACKNOWLEDGED`, `INVESTIGATING`, `RESOLVED`, `FALSE_POSITIVE`.  
Every transition records actor, timestamp, previous state, new state, and reason.

## 8. Hybrid response

V1:
- No destructive automated action.
- Suggestions and simulation.
- Sensitive actions require explicit confirmation.
- Actions must be scoped, reversible where possible, and auditable.
- Do not execute arbitrary commands.
- If confidence is low or data is insufficient, do not act automatically.

Possible future actions, to be assessed one by one: temporarily disabling a local rule, quarantining an item only with a safe OS integration, or guiding the user through a manual procedure. None is enabled by default.

## 9. IoT

V1 may expose passively observed inventory and information available from the system/network. Do not perform intrusive scans. Explicitly report:
- data source;
- last observation;
- completeness level;
- unsupported routers/protocols;
- uncertainty about identification.

## 10. Detection tests

| ID | Scenario | Expected result |
|---|---|---|
| DET-01 | Fixture matching a rule | Detection expected |
| DET-02 | Similar benign fixture | No unjustified detection |
| DET-03 | Incomplete event | Controlled handling |
| DET-04 | Duplicate event | Dedup according to policy |
| DET-05 | Correlated events | One incident with correct references |
| DET-06 | Uncorrelated events | Separate incidents |
| DET-07 | Disabled rule | No detection from that rule |
| DET-08 | Updated rule version | Version recorded in the result |
| DET-09 | Low confidence | No automated action |
| DET-10 | False positive feedback | Audit and updated status |
| DET-11 | Response simulation | No system change |
| DET-12 | Sensitive action without approval | Blocked |
| DET-13 | Approved action | Only the permitted action, audited |
| DET-14 | Restart during the pipeline | No unhandled loss/corruption |
| DET-15 | Regression dataset | Metrics calculated and comparable |

## 11. Metrics

Precision, recall, F1, FP per 1,000 events, ingest-to-detection time, percentage of unclassifiable events, and coverage per source. Report dataset limitations and versions.

## Architectural update 1.1 – consolidated decisions (2026-11-15)

The following decisions supersede any earlier wording in this document:

- **Project license:** MIT. The full text is in the `LICENSE` file at the repository root. Dependency licenses must still be verified and respected separately.
- **Repository:** GitHub monorepo, with code, documentation, tests, and build configurations in the same repository.
- **Desktop dashboard:** Tauri + React + TypeScript. The Rust runtime inside Tauri is distinct from the HSOC specialist Rust modules.
- **Application stack:** Go + Rust + Python. Go is the core and the sole authority for writing to SQLite; Rust implements specialist security components; Python implements workers and auxiliary analysis tasks. Responsibilities must remain separate and process/module boundaries documented.
- **Gradual introduction:** the three languages are part of the approved stack, but their components are integrated in verifiable vertical phases; no empty folders or modules are created, and a worker must not become a single point of failure for the core.
- **Docker:** optional, reserved for development/test environments or isolated auxiliary services. It is not a prerequisite for installation or operation for the HSOC user.
- **Remote agents:** excluded from V1; design and functional implementation from V2+. V1 may define versioned contracts and schemas to avoid a future redesign, without opening remote listeners.
- **Initial platform:** Windows x86-64; Linux and macOS remain subsequent evolutions.
- **Traceability:** requirements, documents, components, tests, and evidence must be linked in the Document 14 matrix.

These decisions update the design baseline; they do not attest that code, CI workflows, installers, or tests have already been implemented or verified.

### Language responsibilities in the detection pipeline

Go maintains the pipeline, state, active rules, persistence, and audit. Rust may provide specialist security functions through narrow interfaces. Python may perform auxiliary analysis and non-authoritative worker tasks. No result produced by Rust/Python is considered trustworthy without Go validation. Workers do not directly apply actions to the system and do not write to SQLite.
