# HSOC — Deployment & Installation

**Version:** 1.1-draft | **Initial target:** Windows x86-64

## 1. Objective

Define a repeatable installation of the Go service and the desktop dashboard. The initial version requires neither external hardware nor a cloud account.

## 2. Platform

- First platform: Windows x86-64.
- Supported Windows versions: **TO BE CONFIRMED** with a tested matrix.
- Linux/macOS: V2+ after core portability.
- Do not claim support for untested OS versions.

## 3. Distributed components

- HSOC Go service.
- Tauri dashboard.
- Initial configuration.
- SQLite schema/migrations.
- Versioned detection rules.
- Documentation, licenses, and SBOM.
- Updater only when signing and rollback are ready.

## 4. Logical directories

The final paths must follow Windows conventions and ACLs:
- binaries installed in a protected directory;
- configuration in a dedicated area;
- database and runtime data in ProgramData or a defined user directory;
- logs separated from secrets;
- backups in a user-chosen destination;
- temporary caches separate from persistent data.

Do not hardcode absolute paths; use OS APIs and test ACLs.

## 5. Installation

1. Verify the package signature and integrity.
2. Check OS and architecture.
3. Create directories with minimal ACLs.
4. Install binaries and policy files.
5. Initialize DB and migrations.
6. Register the service with a minimal account/privileges.
7. Start the service and check health.
8. Start the UI and verify IPC.
9. Show initial status and coverage limits.
10. Do not enable Telegram or auto-response without opt-in.

## 6. Update

- Signed and verified package.
- Pre-update backup.
- Space and compatibility check.
- Controlled DB migration.
- Post-update health verification.
- Binary rollback and DB restore if needed.
- No silent update that changes sensitive policies.

## 7. Uninstallation

- Stop the service.
- Remove service registration and binaries.
- User data kept by default, with an explicit option to remove it.
- Before deleting DB/backups, show the consequences and ask for confirmation.
- Remove credentials from the Credential Store.
- Record the outcome; do not leave orphan processes.

## 8. Recovery

- If the service does not start, the UI shows instructions and a correlation ID.
- If the DB is invalid, do not overwrite it automatically.
- Offer restore from a verified backup.
- Keep a copy of the problematic DB only if the user authorizes it and with adequate protection.

## 9. Deployment tests

| ID | Test | Expected result |
|---|---|---|
| DEP-01 | Clean installation | Service and UI installed |
| DEP-02 | Unsupported OS | Installation blocked with message |
| DEP-03 | Invalid package signature | Installation blocked |
| DEP-04 | Directory ACL | Access restricted |
| DEP-05 | First start | DB initialized, health OK |
| DEP-06 | Windows restart | Service recovers according to policy |
| DEP-07 | UI closed | Service continues without crashing |
| DEP-08 | Update N-1 → N | Data preserved and health OK |
| DEP-09 | Interrupted update | Documented recovery/rollback |
| DEP-10 | Insufficient disk space | Error handled |
| DEP-11 | Uninstallation keeps data | Data present and service removed |
| DEP-12 | Uninstallation with data removal | Only after confirmation |
| DEP-13 | Reinstallation with an existing DB | No unauthorized overwrite |
| DEP-14 | Installation without additional hardware | Core functions operational |

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

### Language packaging and runtimes

The Windows distribution must include the runtime components needed by the user without requiring separate installations of Go, Rust, Python, Node.js, or Docker. Packaging will need to bundle the Go executable, the compiled Rust modules, and the Python workers with the chosen runtime/deployment and their pinned dependencies. The Tauri dashboard distributes the compiled React/TypeScript assets.

The concrete packaging mode for the Python workers and the possible use of separate processes are technical decisions to be validated with a prototype. Docker is not included as a requirement for user installation.
