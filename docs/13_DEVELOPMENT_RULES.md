# HSOC — Mandatory Development Rules

**Version:** 1.1-draft | **Applies to:** every contribution and change

## 1. Architectural rules

1. Go is the central authority of V1.
2. Only Go writes to SQLite.
3. The UI communicates with Go via documented IPC.
4. No optional module accesses the DB directly.
5. The approved stack includes Go, Rust, and Python: Go is the core, Rust the specialist security modules, and Python the workers/auxiliary analysis. Integration happens in phases and with tested contracts.
6. Remote agents are not part of V1.
7. Do not expose network ports by default.
8. Do not introduce mandatory external hardware.

## 2. Security rules

1. Least privilege in every process.
2. Every external input is untrusted.
3. Validate type, size, encoding, and range.
4. Do not execute shell commands derived from input.
5. Do not store secrets in plaintext in config, DB, logs, or the repository.
6. Use the operating system Credential Store.
7. Redact tokens and sensitive data in logs.
8. Telegram default OFF.
9. Auto-response default OFF.
10. Sensitive operations authorized, confirmed, and audited.
11. No unverified updates.
12. Fail-safe: on error, do not apply risky actions.

## 3. Go rules

- Format with `gofmt`.
- Use error wrapping and context.
- Handle context cancellation and timeouts.
- Avoid goroutines without deterministic shutdown.
- Do not use mutable global state without justification.
- Single repository layer for SQLite.
- Transactions for multi-entity changes.
- Parameterized queries.
- Limits on payloads, batches, and concurrency.
- Unit tests for logic and failure paths.

## 4. Rust rules

Applicable to all Rust modules in the project:
- `cargo fmt --check`, `cargo clippy -- -D warnings`, and `cargo test`;
- explicit error handling;
- narrow, versioned FFI/IPC boundary;
- no DB access;
- no implicit additional privileges;
- `cargo audit` and dependency inventory;
- tests on malformed input and panic boundaries.

## 5. Python rules

Applicable to all Python workers and modules in the project:
- isolated environment and lockfile;
- `ruff`, `pytest`, and `pip-audit`;
- no direct DB access;
- no secrets in notebooks/logs;
- no automatic execution of generated scripts;
- schema-validated input/output;
- Python must not prevent the core from working.

## 6. UI rules

- No direct SQLite queries.
- No implicit trust in displayed data.
- Content escaping and injection prevention.
- Handling of loading/error/empty/offline states.
- Confirmation for destructive operations.
- Show severity separately from confidence.
- Explain sources, limits, and rationales.
- No secrets in local storage or UI logs.

## 7. Database rules

- Numbered, versioned, and tested migrations.
- FKs enabled.
- Parameterized queries.
- WAL-aware backup.
- No silent deletion.
- Documented and audited retention.
- Integrity tests after restore.
- DB file and directory with minimal ACLs.

## 8. Detection rules

- Every rule has an ID and version.
- Every rule has positive and negative fixtures.
- Readable rationale for every detection.
- Confidence and severity kept distinct.
- False positive feedback does not delete evidence.
- Every rule change runs a regression.
- No automated response in the MVP.
- Metrics per category, not only aggregate.

## 9. Process rules

Every change follows:
1. requirement/bug and ID;
2. impact analysis and threat review;
3. design/ADR if needed;
4. small implementation;
5. unit tests;
6. applicable integration/security tests;
7. review;
8. documentation and matrix update;
9. merge only with green CI;
10. evidence associated with the release.

## 10. Git and review rules

- Descriptive branches and commits.
- Small, focused PRs.
- No secrets or user data in the repository.
- No generated binaries committed without reason.
- At least one review for changes to security, storage, IPC, or response.
- Every exception to the rules must be justified and recorded.

## 11. Dependency rules

- Add a dependency only with justification.
- Prefer maintained libraries with a compatible license.
- Pin versions through lockfiles.
- Run vulnerability scans.
- Remove unused dependencies.
- Do not auto-update in production without tests.

## 12. PR checklist

- [ ] Requirement/bug linked.
- [ ] Threat/permission impact assessed.
- [ ] No DB access outside Go storage.
- [ ] No secret introduced.
- [ ] Error handling and redacted logging.
- [ ] Unit tests added/updated.
- [ ] Integration/security tests run when required.
- [ ] No regression in previous tests.
- [ ] Documents and matrix updated.
- [ ] CI green.
- [ ] Limits and open decisions recorded.

## 13. Blocking violations

Block merge/release if the change:
- exposes secrets;
- allows unauthorized IPC access;
- executes arbitrary commands;
- expands privileges without justification;
- enables unapproved auto-response;
- bypasses validation;
- writes to the DB from the UI/worker;
- deletes data without audit/consent;
- distributes unverified updates.

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

### Repository and license rules

- The project uses the MIT license, kept in the root file `LICENSE`.
- Every new dependency must have a verified license and, if necessary, attribution in `THIRD_PARTY_NOTICES.md`.
- The repository is organized as a monorepo.
- The dashboard uses Tauri + React + TypeScript.
- Docker is allowed for development/testing, but cannot become a prerequisite for the end user.
- GitHub Actions workflows with minimal permissions; no secrets exposed to untrusted Pull Request workflows.
- `main` is protected by Pull Requests and mandatory CI checks, according to the available features.
- V2 contracts for remote agents may be documented, but listeners and remote agents are not implemented in V1.
