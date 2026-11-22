# HSOC — Complete Master Summary

**Version:** 1.1-draft | **Date:** 2026-11-15  
**Purpose:** single orientation document for development, testing, and maintenance.

## 1. Vision

HSOC is a home cybersecurity center, desktop and local-first. V1 primarily protects and observes the computer it is installed on; IoT visibility is limited to data accessible without additional hardware and without intrusive scans.

Objective: make inventory, events, detections, incidents, status, and backups available in an understandable interface, without mandatory server/cloud costs.

## 2. Confirmed constraints

- Target cost: €0 for basic operation.
- No external hardware required.
- No mandatory camera or sensor.
- No dedicated HSOC login in V1.
- Desktop, mixed mode: background service + on-demand dashboard.
- Windows first; Linux/macOS later.
- A single main agent in V1; remote agents in V2+.
- Conservative hybrid response.
- Least privilege.
- Local data with backup.
- Optional Telegram.
- Monthly vulnerability checks and off-cycle checks for critical issues.
- Auto-fix only as a proposed patch, tested and reviewed by a person.

## 3. Functionality

### Core
- Service lifecycle and health monitor.
- Collection of supported OS events.
- Validation and normalization.
- SQLite storage.
- Versioned detection rules.
- Dedup and correlation.
- Incident management.
- Dashboard.
- Audit and diagnostics.

### Security and maintenance
- Software inventory.
- Vulnerability checks.
- Dependency scan and SBOM.
- Backup/restore.
- Update signing and verification.
- Validated config and policy.
- Detection tests and metrics.

### Optional
- Telegram, disabled initially.
- Specialist Rust security modules and Python auxiliary analysis workers, integrated in phases.
- Response simulation.

### Future V2+
- Remote agents.
- Enrollment, identity, and revocation.
- Coordinator and sync.
- Linux and macOS.
- Distributed policies.

## 4. Architecture

```text
Tauri + React + TypeScript
        |
        | authorized local IPC
        v
HSOC Service (Go)
  |-- service/lifecycle/health
  |-- collector + normalizer
  |-- inventory + vulnerability
  |-- detection + incidents
  |-- response + approvals
  |-- config + policy
  |-- audit + diagnostics
  |-- notifications
  |-- backup/restore
  |
  v
SQLite (Go exclusive writer)

OS adapters: Windows first
OS Credential Store: secrets
Rust: specialist security modules; Python: workers/auxiliary analysis; both isolated and without direct DB access
```

## 5. Language stack

| Language/technology | Role | Mandatory |
|---|---|---|
| Go | Core and service | Mandatory |
| Tauri | Desktop shell | Mandatory for the desktop UI |
| React | UI | Mandatory in the proposal |
| TypeScript | Typed UI | Mandatory in the proposal |
| SQLite | Database | Mandatory in V1 |
| Rust | Specialist security modules | Approved language; phased integration |
| Python | Workers and auxiliary analysis | Approved language; phased integration |
| SQL | Schema and queries | Required for storage |

Rule: do not add runtimes just as a preference. Every extra language must solve a requirement and have a justified packaging/maintenance cost.

## 6. Repository structure

```text
HSOC/
├── .github/workflows/       # CI Go/Rust/Python/dashboard, security, release
├── cmd/                     # Go entry points
├── internal/                # Go core and application modules
├── rust/                    # specialist security modules
├── python/                  # workers and auxiliary analysis
├── desktop/
│   ├── src/                 # React + TypeScript
│   └── src-tauri/           # Tauri shell
├── database/                # schema, migrations, queries
├── protocols/               # IPC, events, messages, and schemas
├── rules/                   # detection, correlation, IoT, vulnerability
├── configs/                 # defaults and platform configurations
├── installers/windows/      # initial packaging
├── scripts/                 # build, test, development, and release
├── tests/                   # unit, integration, system, security, recovery
├── docker/                  # optional: development/testing
├── docs/                    # documentation and ADRs
├── README.md
├── LICENSE                  # MIT
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
└── go.sum
```

V1 is local and Windows-first. Folders are created when the corresponding modules are implemented; no empty directories are added. Remote agents are a V2+ feature.

## 7. Essential development rules

1. Go is the central authority.
2. Only Go writes to SQLite.
3. The UI communicates via IPC and does not read DB files.
4. IPC protected by ACLs and per-operation authorization.
5. Least privilege.
6. Input always validated.
7. No arbitrary commands from input or notifications.
8. Secrets only in the Credential Store.
9. Telegram and auto-response OFF by default.
10. Remote agents excluded from V1.
11. Every detection rule has positive and negative tests.
12. Every change updates tests, documentation, and the matrix.
13. Green CI and review before merging.
14. No PASS claim without evidence.

Consult `13_DEVELOPMENT_RULES.md` for the complete rulebook.

## 8. Test strategy

- Unit: functions, parsers, rules, config, repositories.
- Integration: IPC, DB, migrations, mock notifications.
- System: end-to-end flows and UI.
- Security: ACLs, input, secrets, updates, least privilege.
- Recovery: backup, restore, crash, disk full.
- Performance: CPU/RAM, throughput, latency, and DB growth.
- Detection quality: TP/FP/TN/FN, precision, recall, F1.
- Deployment: install, upgrade, rollback, uninstall.

A test that is defined but not executed is `NOT RUN`. The result must be tied to a commit, version, environment, and evidence.

## 9. MVP vertical slice

1. Event fixture.
2. Go collector/ingest.
3. Normalization.
4. SQLite write.
5. Demo rule.
6. Detection and incident.
7. IPC.
8. Dashboard.
9. Restart and integrity verification.

No auto-response. The first milestone is to demonstrate a complete, repeatable flow, not to implement every feature immediately.

## 10. Roadmap

### Phase A — Foundations
Repository, CI, ADR decisions, logging, config, DB schema, service lifecycle.

### Phase B — End-to-end MVP
Minimal IPC, collector, normalizer, storage, demo detection, incident, UI.

### Phase C — Robust V1
Dedup, correlation, inventory, vulnerability checks, backup/restore, audit, diagnostics, installer.

### Phase D — Extended V1
Telegram opt-in, simulation, detection improvements, release pipeline, and SBOM.

### Phase E — V2+
Linux/macOS, remote agents, identity, enrollment/revocation, sync, and coordination.

## 11. Mandatory release tests

- P0 tests passed.
- Applicable security tests passed.
- Installation on supported OS versions.
- Updates and rollback verified.
- Backup and restore demonstrated.
- No secrets in the repo/logs.
- Dependencies scanned.
- SBOM generated.
- Known issues documented.
- Traceability matrix updated.
- No open critical bug without risk acceptance.

## 12. Still-open decisions

- Named Pipes and the IPC ACL model.
- Minimum supported Windows versions.
- Final SQL schema and dedup/null handling.
- Config format and precedence.
- Approvals/actions relationships.
- Retention and storage budget.
- Backup protection/encryption.
- Performance budget on the reference machine.
- Windows event sources to include in the MVP.
- FFI/IPC boundaries and packaging of the Rust modules.
- Isolation, packaging, and error handling of the Python workers.

## 13. Additional useful proposals

The following are proposals, not already approved requirements:

1. **Demo mode with a synthetic dataset:** allows exploring the dashboard and detections without collecting real data.
2. **Health score for coverage, not for "absolute security":** indicate active sources, latest acquisitions, missing checks, and data quality; avoid a score that suggests guaranteed protection.
3. **Explainability for every alert:** "why do I see this", data used, rule/version, and what has not been verified.
4. **Change journal:** timeline of configurations, updates, rules, and restores.
5. **Export/import of signed rules:** useful for distributing rule sets without modifying the core, with signature, version, and tests.
6. **Automated backup test on an isolated copy:** creating the backup is not enough; periodically verify that it can be restored.
7. **Configurable resource budget:** limits on CPU, RAM, log size, and DB space, with safe degraded behavior.
8. **Privacy dashboard:** shows what stays local, what is exported, and which integrations are active.
9. **Transparent router compatibility:** catalog only tested sources/integrations, without promising universal visibility.
10. **Canary release and rollback:** reduces the risk of faulty updates.
11. **Versioned regression dataset:** makes it possible to measure whether a change improves or worsens false positives.
12. **UI accessibility:** keyboard navigation, contrast, textual descriptions, and messages not based on color alone.

## 14. Main risks

- Limited visibility on consumer IoT/routers.
- False positives and false negatives.
- Excessive privileges for OS sources.
- Complexity of updates and migrations.
- Backup that appears valid but cannot be restored.
- Increased attack surface from multiple runtimes.
- Dependency on external services for notifications.

Mitigate with a realistic scope, reproducible tests, secure defaults, and documented limits.

## 15. Concluding rule

HSOC quality is not measured by the number of features, but by the correctness of the data flow, the security of the boundaries, the explainability of detections, the ability to recover, and verifiable evidence. Every release must demonstrate what it claims.

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

### Final decisions added in revision 1.1

- MIT license.
- GitHub monorepo.
- Tauri + React + TypeScript dashboard.
- Go + Rust + Python stack: Go core and sole SQLite writer; Rust specialist security; Python workers/auxiliary analysis.
- Docker optional, development/testing only.
- Remote agents implemented from V2, not in V1.
- Document 16 dedicated to GitHub, CI/CD, and DevSecOps.
