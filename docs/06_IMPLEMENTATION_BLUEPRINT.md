# HSOC — Implementation Blueprint

**Version:** 1.1-draft | **Objective:** translate the specifications into implementable modules

## 1. Reference stack

- Go: service, IPC, policy, collection, normalization, detection, storage, backup, and audit.
- Tauri + React + TypeScript: desktop UI.
- SQLite: local storage, writable only by Go.
- Rust: specialist security modules, with an explicit FFI/IPC boundary.
- Python: workers and auxiliary analysis tasks, with schema-validated input/output.
- CI: distinct workflows for Go, Rust, Python, and the dashboard, on the components present.

Go + Rust + Python are the approved stack. Integration is gradual and requirement-driven; each runtime has distinct responsibilities and a packaging/maintenance plan.

## 2. Proposed repository structure

```text
hsoc/
├── README.md
├── LICENSE
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
├── go.sum
├── cmd/
│   └── hsoc-service/
│       └── main.go
├── internal/
│   ├── service/
│   ├── ipc/
│   ├── config/
│   ├── collector/
│   ├── normalizer/
│   ├── inventory/
│   ├── detection/
│   ├── incidents/
│   ├── response/
│   ├── storage/
│   ├── backup/
│   ├── notifications/
│   ├── audit/
│   └── diagnostics/
├── rust/           # specialist security modules
├── python/         # workers and auxiliary analysis
├── database/
│   ├── schema/
│   ├── migrations/
│   └── queries/
├── protocols/      # IPC, events, messages, and schemas
├── desktop/
│   ├── src/        # React + TypeScript
│   └── src-tauri/  # Tauri shell, distinct from rust/
├── tests/
│   ├── fixtures/
│   ├── integration/
│   ├── system/
│   ├── security/
│   └── recovery/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   └── operations/
├── scripts/
└── .github/workflows/
```

## 3. Module dependencies

- `cmd` starts `service`.
- `service` coordinates internal modules.
- `ipc` exposes only authorized use cases.
- `storage` is the only layer that talks to SQLite.
- `collector` does not decide severity; it produces raw/normalized data.
- `normalizer` validates and maps canonical events.
- `detection` evaluates rules on normalized events.
- `incidents` aggregates detections.
- `response` applies policy and approvals; in the MVP, simulation only.
- `audit` records sensitive changes.
- `ui` talks only via IPC.

Avoid cyclic dependencies and global DB access.

## 4. Mandatory vertical slice

Before implementing many features, build a single complete and secure chain:

1. acquire a test event from a fixture or a non-destructive OS source;
2. validate and normalize it in Go;
3. store it in SQLite through a Go repository;
4. evaluate it with a deterministic demo rule;
5. create a detection record and an incident;
6. expose the data via IPC;
7. display it in the dashboard;
8. record logs and audit where necessary;
9. test restart and recovery.

No automated response in this slice.

## 5. Implementation rules

- Small, testable interfaces.
- Context/cancellation and timeouts for I/O.
- Typed errors and redacted messages.
- Config validated at startup and on every update.
- No shell execution derived from input.
- No goroutines without a lifecycle/stop path.
- DB access only through repositories.
- Migrations separate from application code.
- Synthetic, reproducible fixtures.
- Feature flags for unfinished functions.
- Dependencies pinned to versions and updated through PRs.

## 6. Language strategy

**Go**: V1 core, orchestration, policy, persistence, and sole SQLite writer.  
**Rust**: specialist security modules, with narrow interfaces, tests, dependency audit, and verified packaging.  
**Python**: workers and auxiliary analysis; schema-validated input/output, pinned environment and dependencies, isolation, and failure handling. It is not the authority over state or the database.

## 7. Minimum CI

1. Go: `gofmt` check, `go vet`, `go test ./...`, `govulncheck`.
2. Rust: `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test`, `cargo audit`.
3. Python: `ruff check`, `ruff format --check`, `pytest`, `pip-audit`.
4. Dashboard: lint, TypeScript typecheck, tests, and build.
5. Integration tests with a temporary DB and migrations.
6. Dependency vulnerability scan and secret scan.
7. Reproducible build and reports associated with the commit.

## 8. Implementation sequence

1. Repo, logging, config, and build.
2. DB schema and migrations.
3. Service lifecycle and health.
4. IPC and authorization.
5. First collector and normalizer.
6. Detection rule engine.
7. Incidents and audit.
8. Dashboard.
9. Backup/restore.
10. Inventory and vulnerability checks.
11. Opt-in notifications.
12. Hardening, installer, and release.

## 9. Implementation tests

| ID | Area | Expected outcome |
|---|---|---|
| IMP-01 | Core build | Reproducible build |
| IMP-02 | Isolated modules | Unit tests pass |
| IMP-03 | Dependencies | No architectural cyclic imports |
| IMP-04 | DB access | Only the storage package opens the DB |
| IMP-05 | UI | No direct access to DB files |
| IMP-06 | Invalid config | Fail safe |
| IMP-07 | Lifecycle | Clean stop without leftover goroutines |
| IMP-08 | CI | Pipeline fails on test/security gate |
| IMP-09 | Fixtures | Same input produces the same result |
| IMP-10 | Missing worker | Core degrades in a controlled way |

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

### Reference monorepo structure

```text
HSOC/
├── .github/workflows/       # CI Go/Rust/Python/dashboard, security, release
├── cmd/                     # Go entry points
├── internal/                # Go core and application modules
├── rust/                    # specialist security modules
├── python/                  # workers and auxiliary analysis
├── desktop/                 # Tauri + React + TypeScript
│   └── src-tauri/           # Tauri shell; distinct from rust/
├── database/                # schema, migrations, queries
├── protocols/               # IPC, events, messages, and schemas
├── rules/                   # detection, correlation, IoT, vulnerability
├── configs/                 # defaults and platform configurations
├── installers/windows/      # initial packaging
├── scripts/                 # build, test, development, and release
├── tests/                   # unit, integration, system, security, recovery
├── docker/                  # optional development/test environments only
├── docs/                    # documentation and ADRs
├── README.md
├── LICENSE                  # MIT
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
└── go.sum
```

Component folders are created when their module starts, not as empty directories without responsibilities. V2 remote agents will have a dedicated future section and must not be implemented in V1.
