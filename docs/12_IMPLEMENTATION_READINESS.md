# HSOC — Consolidation & Implementation Readiness

**Version:** 1.1-draft | **Status:** design gate before implementation

## 1. Objective

Verify that the specifications are coherent enough to start development without critical ambiguity. This document does not declare that the code has already been written.

## 2. Mandatory principles

- Local-first and minimal cost.
- No mandatory external hardware.
- Go as central authority.
- SQLite written only by Go.
- UI separated from the service.
- Least privilege.
- Privacy and data minimization.
- Validated and versioned config.
- Testable backup/restore.
- Requirement-test traceability.
- Remote agents excluded from V1.
- Auto-response disabled.
- IoT observed passively.

## 3. Decisions to consolidate

| ADR | Topic | Proposal |
|---|---|---|
| ADR-121 | IPC | Windows Named Pipes with ACL; validate client identity |
| ADR-122 | Rust | Approved language for specialist security modules; define FFI boundaries and packaging |
| ADR-123 | Python | Approved language for workers and auxiliary analysis; define isolation and packaging |
| ADR-124 | Auto-response | OFF in V1; explicit approval |
| ADR-125 | Config | Versioned schema and defined precedence |
| ADR-126 | Source of truth | DB for runtime state; config for bootstrap/preferences |
| ADR-127 | Health monitor | Status per component and degradation |
| ADR-128 | Diagnostics | Voluntary and redacted export |
| ADR-129 | Simulation | Mode with no effect on the system |
| ADR-130 | Remote agents | Excluded from V1 |
| ADR-131 | Architecture | Initial Windows x86-64 |
| ADR-132 | Supported Windows versions | Define and test the OS matrix |

## 4. Definition of Ready

A feature is ready for development if:
- the requirement and rationale are defined;
- version/priority are assigned;
- normal behavior and failure modes are described;
- processed data and privacy are clear;
- threats and authorizations are assessed;
- interfaces and schema are defined;
- positive, negative, and recovery tests are written;
- dependencies and limits are known;
- documents and the traceability matrix are updated.

## 5. Definition of Done

A feature is complete only when:
- implementation and review are finished;
- unit/integration tests pass;
- applicable security tests pass;
- logging/error handling are adequate;
- no secret is exposed;
- documentation is updated;
- the requirement is linked to tests and evidence;
- known limits are recorded;
- a feature flag/safe default is set;
- the build is reproducible.

## 6. First vertical slice

Synthetic/non-destructive OS event → Go collector → validation/normalization → SQLite → demo rule → detection → incident → IPC → dashboard.

Criteria:
- reproducible fixture;
- a single end-to-end path;
- no automated actions;
- intact DB after restart;
- the UI does not access the DB;
- automated tests and attached reports.

## 7. Test plan per gate

- Foundations: unit tests, config, logging, lifecycle.
- Storage: migrations, FKs, transactions, dedup.
- IPC: authorization, schema, timeout, rate.
- Detection: positive/negative fixtures, metrics.
- UI: states, errors, accessibility.
- Recovery: backup, restore, crash, disk full.
- Security: ACLs, secrets, updates, least privilege.
- Release: installer, upgrade, rollback, SBOM.

## 8. Development sequence

1. Repository, standards, CI, and documentation.
2. Data model and migrations.
3. Service lifecycle/health.
4. IPC.
5. Collector/normalizer.
6. Detection/incident.
7. Dashboard.
8. Backup/restore.
9. Inventory/vulnerability checks.
10. Opt-in notifications.
11. Hardening and installer.
12. Release candidate.

## 9. Blocking risks

- IPC without robust authorization.
- DB writable by multiple components.
- No tested restore.
- Service with excessive privileges.
- Unverified updates.
- Unapproved automated actions.
- Overestimated IoT telemetry.
- P0 requirements without tests.

## 10. GO/NO-GO

**GO:** coherent architecture, closed P0 decisions, repeatable vertical slice, P0 tests passed, verified recovery.  
**NO-GO:** data loss, unauthorized access, exposed secrets, unauthorized actions, failed restore, or missing critical tests.

## 11. Recommended first sprint

- Create the repository and CI.
- Freeze the canonical event schema and the first DB schema.
- Prototype the service lifecycle and health.
- Define and test minimal IPC.
- Implement the vertical slice with fixtures.
- Produce the first test report and update the matrix.

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

### Updated readiness gate

The choice of the Go + Rust + Python stack and the Tauri + React + TypeScript dashboard is confirmed. Before starting each module, the contract, input/output, failure modes, dependencies, packaging, and tests must still be defined. The fact that a language is approved does not authorize a module without a requirement or a clear responsibility.

Docker remains optional for development/testing. V1 remains local and Windows-first; remote agents only from V2.
