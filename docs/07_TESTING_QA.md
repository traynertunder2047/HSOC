# HSOC — Testing & Quality Assurance

**Version:** 1.1-draft | **Status:** test plan, not an execution report

## 1. Objective

Demonstrate with repeatable evidence that every implemented requirement works, that regressions are caught, and that the software handles errors and recovery safely.

## 2. Test levels

- **Unit:** functions, parsers, normalizer, scoring, config, repositories.
- **Integration:** DB, IPC, migrations, mock notifications, backup.
- **System:** installation, service, UI, end-to-end flows.
- **Security:** ACLs, authorization, input validation, secrets, updates.
- **Reliability:** crash, restart, disk full, DB busy, backup/restore.
- **Performance:** CPU, RAM, I/O, DB growth under synthetic load.
- **Detection quality:** precision, recall, false positives and false negatives on a labeled dataset.
- **Compatibility:** supported OS versions and UI/service combinations.
- **Acceptance:** MVP criteria and release gates.

## 3. Mandatory test case

Every test must have:
- a stable ID;
- a linked `TR-*` requirement;
- title and objective;
- version/commit;
- prerequisites and environment;
- fixture/input;
- reproducible procedure;
- expected result;
- actual result;
- evidence (logs, screenshots, reports);
- status: `NOT RUN`, `PASS`, `FAIL`, `BLOCKED`, `SKIPPED`;
- author and date.

## 4. Minimum catalog

### Core and integration
- INT-01: service startup and health.
- INT-02: collector → normalizer.
- INT-03: normalizer → storage.
- INT-04: detection → incident.
- INT-05: IPC → UI.
- INT-06: valid config reload.
- INT-07: invalid config reload.
- INT-08: audit of a sensitive operation.
- INT-09: optional worker missing.
- INT-10: recovery after UI disconnection.

### Detection
- DET-01..05: positive rules with known fixtures.
- DET-06..08: similar but benign negative events.
- DET-09: dedup.
- DET-10: correlation.
- DET-11: severity.
- DET-12: confidence.
- DET-13: rule version.
- DET-14: incomplete/malformed event.
- DET-15: regression on a synthetic historical dataset.

### Configuration
- CFG-01..05: defaults and schema.
- CFG-06..08: precedence and overrides.
- CFG-09: mandatory policy that cannot be disabled.
- CFG-10: atomic reload.
- CFG-11: config rollback.
- CFG-12: secrets in the credential store.
- CFG-13: config backup/restore.
- CFG-14: invalid input.
- CFG-15: audit of changes.

### Deployment
- DEP-01..14: refer to Document 05; keep evidence for every tested OS/build.

### Recovery
- REC-01: full backup.
- REC-02: wrong checksum.
- REC-03: restore onto a test DB.
- REC-04: interrupted restore.
- REC-05: recovery after a crash.
- REC-06: failed migration.
- REC-07: read-only DB.
- REC-08: insufficient space.

## 5. Detection metrics

Per category and threshold:
- TP: true positives;
- FP: false positives;
- TN: true negatives;
- FN: false negatives;
- Precision = TP / (TP + FP);
- Recall = TP / (TP + FN);
- F1 = harmonic mean of precision and recall.

Report dataset support, period, rule version, threshold, and limitations. Do not use a single aggregate score to hide weak categories. Do not optimize only alert reduction: a system that reports nothing can have few FPs but also many FNs.

## 6. Quality criteria

- No unhandled crash in regression tests.
- No open critical/high defect without formal risk acceptance.
- Backup and restore passed.
- All P0 tests passed or blocked with an approved justification.
- No secrets found by secret scanners.
- No unmitigated critical vulnerability in dependencies.
- Performance within budgets defined before benchmarking.
- Every P0 requirement has at least one associated test.

## 7. Performance tests to be defined

Measure the baseline on declared reference hardware:
- idle CPU/RAM of the service;
- CPU peak during ingest;
- synthetic event throughput;
- ingest-to-dashboard latency;
- DB size per day/volume;
- backup/restore time;
- behavior with a large DB.

Do not set arbitrary thresholds before having a realistic baseline.

## 8. Recommended CI

- Go: `gofmt` check, `go vet`, `go test ./...`, `govulncheck`.
- Rust: `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test`, `cargo audit`.
- Python: `ruff check`, `ruff format --check`, `pytest`, `pip-audit`.
- Tauri/React/TypeScript dashboard: lint, typecheck, tests, and build.
- All checks apply to the components actually present in the repository; the plan foresees separate workflows for each.
- Secret scanning: Gitleaks.
- Vulnerability/dependency inventory: OSV-Scanner or an equivalent tool.
- Migration and backup tests in an isolated pipeline.

## 9. Release gate

**GO** if: build, P0 tests, security tests, backup/restore, install/update, and documentation are verified.  
**NO-GO** if: critical tests fail, the update signature cannot be verified, restore does not work, secrets are exposed, or bugs exist that can corrupt data or cause unauthorized actions.

## 10. Test report

Never mark a test PASS without output/evidence. Tests not executed remain NOT RUN. A blocked test is not equivalent to PASS.

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

### Testing multi-language boundaries

Add tests for:
- malformed input and payloads over the limit between Go and Rust;
- Python worker timeouts, crashes, and restarts;
- incompatible schema/version between components;
- incomplete or invalid results returned by a worker;
- confirmation that only Go writes to SQLite;
- the dashboard communicating only through authorized IPC;
- a desktop build without Docker installed.
