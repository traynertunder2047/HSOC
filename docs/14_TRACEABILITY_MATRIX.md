# HSOC — Complete Traceability Matrix

**Version:** 1.1-draft | **Date:** 2026-11-15  
**Note:** planned requirements; the initial states are `TO BE IMPLEMENTED` and `TO BE TESTED`, unless there is verifiable evidence.

## 1. Conventions

- Stable IDs: do not reuse a retired ID.
- Priority: P0 essential for the MVP; P1 important for V1; P2 optional; P3 V2+.
- Versions: V1-MVP, V1, V2+.
- Status: `SPECIFIED`, `TO BE CONFIRMED`, `TO BE IMPLEMENTED`, `TO BE TESTED`, `SUPERSEDED`.
- Every requirement must link to a document, a component, and at least one test.
- Test references are identifiers to be materialized as executable test cases; they do not attest execution.

## 2. Requirements matrix


### 2.1 FUN requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-FUN-001 | Go local agent | 01,06 | `service` | SYS-001,IMP-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-002 | Background service | 01,05,06 | `service` | SYS-001,DEP-06 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-003 | Windows event collection | 01,06,10 | `collector` | INT-02,DET-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-004 | Validation and normalization | 01,06,10 | `normalizer` | INT-02,DET-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-005 | SQLite event persistence | 02,06 | `storage` | DB-001,DB-006 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-006 | Detection rule engine | 01,10 | `detection` | DET-01..15 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-007 | Deduplication | 02,10 | `detection/storage` | DB-005,DET-04 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-008 | Event correlation | 10 | `detection/incidents` | DET-05,DET-06 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-009 | Incident management | 01,10 | `incidents` | INT-04,UI-02 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-010 | Desktop dashboard | 01,03,08 | `ui` | INT-05,UI-01..10 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-011 | Local notifications | 01,08 | `notifications` | INT-08,UI-05 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-012 | Telegram opt-in | 01,08,11 | `notifications` | CFG-10,SEC-05 | P2 | V1 | TO BE IMPLEMENTED |
| TR-FUN-013 | Software inventory | 01,09 | `inventory` | INV-01..05 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-014 | Monthly vulnerability check | 01,09 | `vulnerability` | MNT-01,INV-06 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-015 | Backup and restore | 01,02,05,08 | `backup/storage` | DB-009..14,REC-01..08 | P0 | V1 | TO BE IMPLEMENTED |
| TR-FUN-016 | Configuration and policy | 01,11 | `config` | CFG-01..15 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-FUN-017 | Health monitor | 01,08,12 | `diagnostics` | SYS-001,UI-01 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-018 | Exportable diagnostics | 04,08,12 | `diagnostics` | SEC-09,UI-07 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-019 | Simulation mode | 10,12 | `response` | DET-11 | P1 | V1 | TO BE IMPLEMENTED |
| TR-FUN-020 | Audit of sensitive operations | 02,04,10,11 | `audit` | SEC-12,CFG-12 | P0 | V1-MVP | TO BE IMPLEMENTED |

### 2.2 ARC requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-ARC-001 | Go as central authority | 01,06,12 | `service` | IMP-01,IMP-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-002 | Only Go writes to SQLite | 02,03,06,13 | `storage` | IMP-04,SEC-11 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-003 | Dashboard/service separation | 01,03,06 | `ipc/ui` | INT-05,SEC-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-004 | Authorized local IPC | 03,04 | `ipc` | IPC-001..12 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-005 | Versioned IPC contracts | 03 | `ipc/schemas` | IPC-006 | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-006 | Isolated Python workers for auxiliary analysis, without DB access | 01,02,06,12 | `python/` | IMP-09, QA worker | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-007 | Specialist Rust security modules, without DB access | 01,04,06,12 | `rust/` | IMP-10, SEC tests | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-013 | Tauri + React + TypeScript dashboard | 01,03,06,08 | `desktop/` | UI-01..10 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-014 | GitHub monorepo with CI per stack | 06,07,13,16 | `.github/workflows/` | CI report | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-015 | MIT project license and third-party notices | 06,09,13,16 | `LICENSE`, `THIRD_PARTY_NOTICES.md` | license review | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-016 | Docker optional for development/testing only | 05,06,16 | `docker/` | build without Docker | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-008 | Module lifecycle management | 06 | `service` | IMP-07,IMP-09 | P1 | V1 | TO BE IMPLEMENTED |
| TR-ARC-009 | Remote agents excluded from V1 | 01,12,13 | `scope` | architectural review | P0 | V1 | TO BE IMPLEMENTED |
| TR-ARC-010 | Future versionable protocol | 03 | `schemas/ipc` | IPC-006 | P2 | V2+ | TO BE IMPLEMENTED |
| TR-ARC-011 | Dependencies with controlled direction | 06,13 | `internal packages` | IMP-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-ARC-012 | No remote listener by default | 04,11 | `service/config` | SEC-01,CFG-01 | P0 | V1-MVP | TO BE IMPLEMENTED |

### 2.3 DB requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-DB-001 | Local SQLite | 02 | `storage` | DB-001 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-002 | Foreign keys enabled | 02 | `storage` | DB-004 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-003 | Multi-entity transactions | 02 | `storage` | DB-006 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-004 | Versioned migrations | 02,09 | `migrations` | DB-002,DB-003 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-005 | Integrity after crash | 02,07 | `storage` | DB-013,REC-05 | P0 | V1 | TO BE IMPLEMENTED |
| TR-DB-006 | Fingerprint dedup | 02,10 | `events/detections` | DB-005,DET-04 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DB-007 | Configurable retention | 02,11 | `storage/config` | DB-011,DB-012 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DB-008 | Secrets outside the DB | 02,04,11 | `credential store` | SEC-05,CFG-09 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-009 | Persistent audit | 02,04 | `audit_log` | SEC-12 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-010 | WAL-compatible backup | 02,05 | `backup` | DB-009,REC-01 | P0 | V1 | TO BE IMPLEMENTED |
| TR-DB-011 | UTC timestamps | 02,03 | `storage` | DB-001,INT-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DB-012 | Audited retention deletion | 02,11 | `storage/audit` | DB-011,DB-012 | P1 | V1 | TO BE IMPLEMENTED |

### 2.4 SEC requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-SEC-001 | Least privilege | 01,04,05,13 | `service/installer` | SEC-06,SEC-14 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-002 | No dedicated HSOC login | 01,03 | `ipc` | IPC-001 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-003 | Protected IPC | 03,04 | `ipc` | SEC-01,IPC-002 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-004 | Input validation | 03,04,13 | `all boundaries` | SEC-02..04 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-005 | Secure credential handling | 02,04,08,11 | `credential store` | SEC-05,CFG-09 | P0 | V1 | TO BE IMPLEMENTED |
| TR-SEC-006 | Verified updates | 04,05,09 | `updater` | SEC-08,DEP-03 | P1 | V1 | TO BE IMPLEMENTED |
| TR-SEC-007 | Response authorization | 04,10 | `response` | DET-12,DET-13 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-008 | Audit of sensitive operations | 02,04,10 | `audit` | SEC-12 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-009 | Data minimization | 01,04,08 | `collectors/diagnostics` | SEC-09,UI-07 | P0 | V1 | TO BE IMPLEMENTED |
| TR-SEC-010 | Notifications without excessive data | 04,08,11 | `notifications` | SEC-05,CFG-10 | P1 | V1 | TO BE IMPLEMENTED |
| TR-SEC-011 | Optional module isolation | 04,06 | `worker boundary` | SEC-10,IMP-09 | P1 | V1 | TO BE IMPLEMENTED |
| TR-SEC-012 | Safe DB error handling | 02,04 | `storage` | DB-006..08,SEC-11 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-013 | No arbitrary commands | 03,04,13 | `ipc/response` | SEC-04,IPC-010 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-SEC-014 | Backup protection | 02,04,05 | `backup` | DB-009,REC-02 | P0 | V1 | TO BE IMPLEMENTED |
| TR-SEC-015 | Local-first privacy | 01,08,11 | `all components` | UI-07,CFG-10 | P0 | V1 | TO BE IMPLEMENTED |
| TR-SEC-016 | Future remote agent revocation | 01,04 | `future coordinator` | FUT-004 | P3 | V2+ | TO BE IMPLEMENTED |

### 2.5 DET requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-DET-001 | Separate event/detection/incident | 02,10 | `events/detection/incidents` | INT-04,DET-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-002 | Severity and confidence kept distinct | 08,10 | `detection/ui` | DET-11,DET-12,UI-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-003 | Rationale and evidence | 08,10 | `detection` | DET-01,UI-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-004 | FP/FN tests per rule | 07,10 | `tests/fixtures` | DET-02,DET-15 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-005 | Rule versioning | 02,10 | `rules` | DET-08 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DET-006 | Dedup/correlation | 02,10 | `detection` | DET-04..06 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DET-007 | Incident workflow | 08,10 | `incidents` | INT-04,UI-02 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-008 | Conservative hybrid response | 01,10 | `response` | DET-11..13 | P0 | V1 | TO BE IMPLEMENTED |
| TR-DET-009 | Approval of sensitive actions | 04,10 | `response/approvals` | DET-12,DET-13 | P0 | V1 | TO BE IMPLEMENTED |
| TR-DET-010 | Auto-response OFF by default | 04,10,11 | `response/config` | DET-12,CFG-11 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-011 | Simulation without effects | 10,12 | `response` | DET-11 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DET-012 | Response audit | 02,04,10 | `audit` | SEC-12,DET-13 | P0 | V1 | TO BE IMPLEMENTED |
| TR-DET-013 | Fail-safe in uncertainty | 04,10 | `response` | DET-09,SEC-11 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DET-014 | Detection metrics | 07,10 | `qa/metrics` | DET-15 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DET-015 | Passive and limited IoT | 01,08,10 | `iot observer` | UI-08 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DET-016 | False positive feedback | 08,10 | `incidents/detection` | DET-10 | P1 | V1 | TO BE IMPLEMENTED |

### 2.6 CFG requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-CFG-001 | Secure defaults | 01,11 | `config` | CFG-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-CFG-002 | Schema-validated config | 03,11 | `config` | CFG-02..05 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-CFG-003 | Documented precedence | 11,12 | `config` | CFG-06 | P1 | V1 | TO BE IMPLEMENTED |
| TR-CFG-004 | Non-disableable security constraints | 04,11 | `policy` | CFG-06,SEC-07 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-CFG-005 | Optional modules disabled | 06,11 | `config` | IMP-09,CFG-01 | P1 | V1 | TO BE IMPLEMENTED |
| TR-CFG-006 | Separation of config/DB/secrets | 02,11 | `config/storage` | CFG-09,DB-008 | P0 | V1 | TO BE IMPLEMENTED |
| TR-CFG-007 | Response policy | 10,11 | `response/config` | CFG-11,DET-12 | P0 | V1 | TO BE IMPLEMENTED |
| TR-CFG-008 | Retention | 02,11 | `storage/config` | DB-011 | P1 | V1 | TO BE IMPLEMENTED |
| TR-CFG-009 | Opt-in notifications | 08,11 | `notifications` | CFG-10 | P1 | V1 | TO BE IMPLEMENTED |
| TR-CFG-010 | Preserve valid config | 11 | `config` | CFG-03,CFG-08 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-CFG-011 | Config audit | 02,11 | `audit` | CFG-12 | P0 | V1 | TO BE IMPLEMENTED |
| TR-CFG-012 | OS Credential Store | 04,08,11 | `credential store` | SEC-05,CFG-09 | P0 | V1 | TO BE IMPLEMENTED |
| TR-CFG-013 | Configuration backup | 02,11 | `backup` | CFG-13 | P1 | V1 | TO BE IMPLEMENTED |
| TR-CFG-014 | No V1 remote agent | 01,11,12 | `scope` | review | P0 | V1 | TO BE IMPLEMENTED |
| TR-CFG-015 | Config rollback | 11 | `config` | CFG-14 | P1 | V1 | TO BE IMPLEMENTED |

### 2.7 DEP requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-DEP-001 | Initial Windows x86-64 | 01,05,12 | `installer` | DEP-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-002 | Declared Windows versions | 05,12 | `compatibility matrix` | DEP-02 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-003 | Service/UI installation | 05 | `installer` | DEP-01,DEP-05 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-004 | Directories and ACLs | 04,05 | `installer` | DEP-04,SEC-06 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-005 | DB initialization | 02,05 | `bootstrap` | DEP-05,DB-001 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-006 | Installation health check | 05,12 | `service` | DEP-05,SYS-001 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-007 | Signed updates | 04,05,09 | `updater` | DEP-03,SEC-08 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DEP-008 | Update rollback | 05,09 | `updater/backup` | DEP-09,REC-06 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DEP-009 | Controlled uninstallation | 05 | `installer` | DEP-11,DEP-12 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DEP-010 | No external hardware | 01,05 | `product scope` | DEP-14 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEP-011 | Future Linux | 01,05 | `platform adapters` | FUT-007 | P3 | V2+ | TO BE IMPLEMENTED |
| TR-DEP-012 | Future macOS | 01,05 | `platform adapters` | FUT-008 | P3 | V2+ | TO BE IMPLEMENTED |
| TR-DEP-013 | Reinstallation with existing data | 05 | `installer/storage` | DEP-13 | P1 | V1 | TO BE IMPLEMENTED |
| TR-DEP-014 | Future remote agents | 01,05,12 | `coordinator` | FUT-001..06 | P3 | V2+ | TO BE IMPLEMENTED |

### 2.8 QA requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-QA-001 | Unit tests | 07,13 | `tests/unit` | CI unit | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-002 | Integration tests | 07 | `tests/integration` | INT-01..10 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-003 | System tests | 07 | `tests/system` | SYS-001.. | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-004 | Security tests | 04,07 | `tests/security` | SEC-01..16 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-005 | Regression tests | 07,10 | `tests/regression` | DET-15 | P0 | V1 | TO BE IMPLEMENTED |
| TR-QA-006 | Performance tests | 07 | `tests/performance` | benchmark report | P1 | V1 | TO BE IMPLEMENTED |
| TR-QA-007 | Recovery tests | 02,05,07 | `tests/recovery` | REC-01..08 | P0 | V1 | TO BE IMPLEMENTED |
| TR-QA-008 | Detection quality tests | 07,10 | `tests/detection` | DET-15 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-009 | CI pipeline | 06,07,13 | `workflows` | CI report | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-010 | Definition of Ready | 12,13 | `process` | review checklist | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-011 | Definition of Done | 12,13 | `process` | PR checklist | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-012 | Bug severity classification | 07,09 | `triage` | bug records | P1 | V1 | TO BE IMPLEMENTED |
| TR-QA-013 | Release gates | 07,09,12 | `release` | GO/NO-GO record | P0 | V1 | TO BE IMPLEMENTED |
| TR-QA-014 | Evidence retention | 07 | `test reports` | test artifacts | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-QA-015 | Test environment isolation | 07,12 | `test infra` | environment report | P1 | V1 | TO BE IMPLEMENTED |

### 2.9 MNT requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-MNT-001 | Monthly dependency scan | 09,13 | `CI/maintenance` | MNT-01 | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-002 | Off-cycle critical advisories | 09 | `security process` | MNT-02 | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-003 | SBOM per release | 09 | `release pipeline` | MNT-03 | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-004 | Semantic versioning | 09 | `release` | release review | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-005 | Release channels | 09 | `distribution` | release checklist | P2 | V1 | TO BE IMPLEMENTED |
| TR-MNT-006 | Immutable/versioned migrations | 02,09 | `migrations` | DB-002, MNT-07 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-MNT-007 | Maintained ADRs | 01,12 | `docs/decisions` | ADR review | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-008 | Reviewed patches | 09,13 | `PR process` | MNT-10 | P0 | V1 | TO BE IMPLEMENTED |
| TR-MNT-009 | Known issues and changelog | 08,09 | `release docs` | MNT-08 | P1 | V1 | TO BE IMPLEMENTED |
| TR-MNT-010 | Auto-fix as a reviewed proposal only | 09,13 | `maintenance` | MNT-10 | P0 | V1 | TO BE IMPLEMENTED |

### 2.10 UI requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-UI-001 | Status dashboard | 01,08 | `ui` | UI-01 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-UI-002 | Distinguish events/detections/incidents | 08,10 | `ui` | UI-02 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-UI-003 | Show severity/confidence | 08,10 | `ui` | UI-03 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-UI-004 | Health and diagnostics | 08,12 | `ui` | UI-01,UI-07 | P1 | V1 | TO BE IMPLEMENTED |
| TR-UI-005 | Secure configuration | 08,11 | `ui` | UI-05 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-UI-006 | UI error handling | 03,08 | `ui/ipc` | UI-04 | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-UI-007 | User/technical guides | 08,09 | `docs` | UI-09,UI-10 | P1 | V1 | TO BE IMPLEMENTED |
| TR-UI-008 | Declared IoT limits | 01,08,10 | `ui/docs` | UI-08 | P0 | V1 | TO BE IMPLEMENTED |
| TR-UI-009 | Visible known issues | 08,09 | `ui/docs` | MNT-08 | P1 | V1 | TO BE IMPLEMENTED |
| TR-UI-010 | Restore confirmation and impact | 02,05,08 | `ui/backup` | UI-06 | P0 | V1 | TO BE IMPLEMENTED |

### 2.11 FUT requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-FUT-001 | Remote agents | 01,12 | `future/agents` | architecture review | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-002 | Agent identity | 04 | `future/identity` | security design | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-003 | Authenticated enrollment | 04 | `future/enrollment` | security design | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-004 | Agent revocation | 04 | `future/identity` | security test | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-005 | Synchronization | 01,03 | `future/sync` | integration test | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-006 | Secure remote tasks | 04,10 | `future/tasks` | threat model | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-007 | Linux | 01,05 | `platform/linux` | platform suite | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-008 | macOS | 01,05 | `platform/macos` | platform suite | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-009 | Distributed coordinator | 01 | `future/coordinator` | system test | P3 | V2+ | TO BE IMPLEMENTED |
| TR-FUT-010 | Distributed policies | 11 | `future/policy` | security/integration | P3 | V2+ | TO BE IMPLEMENTED |

### 2.12 Repository and DevSecOps requirements

| ID | Requirement | Doc | Component | Test/evidence | Priority | Version | Initial state |
|---|---|---|---|---|---|---|---|
| TR-DEV-001 | GitHub Actions for Go, Rust, Python, and dashboard | 06,07,13,16 | `.github/workflows/` | CI report | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEV-002 | Protected main branch and mandatory PRs | 13,16 | repository settings | branch protection evidence | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEV-003 | Secret scanning and minimal workflow permissions | 04,13,16 | GitHub settings/workflows | security config review | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEV-004 | Docker not required for user installation | 05,06,16 | installer/build | install test without Docker | P0 | V1-MVP | TO BE IMPLEMENTED |
| TR-DEV-005 | MIT and dependency license inventory | 09,13,16 | LICENSE/SBOM/notices | license review | P1 | V1 | TO BE IMPLEMENTED |

## 3. Cross-cutting tests to maintain

| Test ID | Area | Main requirements | Required evidence |
|---|---|---|---|
| SYS-001..003 | End-to-end | TR-FUN-001..010 | logs, DB fixture, UI screenshot |
| INT-01..10 | Integration | TR-ARC, TR-FUN, TR-DB | automated report |
| SEC-01..16 | Security | TR-SEC-001..016 | security test report |
| DET-01..15 | Detection | TR-DET-001..016 | dataset, metrics, report |
| CFG-01..15 | Configuration | TR-CFG-001..015 | fixtures and result |
| DB-001..14 | Database | TR-DB-001..012 | temporary DB and integrity check |
| DEP-01..14 | Deployment | TR-DEP-001..014 | VM/OS test matrix |
| REC-01..08 | Recovery | TR-FUN-015, TR-DB-005/010 | backup, restore, and checksum |
| UI-01..10 | Usability | TR-UI-001..010 | UI tests and review |
| MNT-01..10 | Maintenance | TR-MNT-001..010 | CI, SBOM, advisory report |
| IMP-01..10 | Implementation | TR-ARC, TR-QA | CI and architecture |

## 4. Test case template

Every test case must contain:
- `test_id`
- `requirement_ids`
- name and objective
- environment, OS, HSOC version, and commit
- prerequisites
- fixture/input
- numbered steps
- expected result
- actual result
- status (`NOT RUN`, `PASS`, `FAIL`, `BLOCKED`, `SKIPPED`)
- date, author, and link/evidence
- associated bugs

## 5. Questions to close before the freeze

1. Full physical SQL specification and indexes.
2. `policy_assignments` and `actions`/`approvals` relationships.
3. UNIQUE/dedup semantics with NULL values.
4. IPC Named Pipes: ACLs, client identity, and protocol.
5. Supported Windows versions.
6. Config/runtime source of truth and config format.
7. Retention and space limits.
8. Credential Store and backup protection.
9. Performance budget on reference hardware.
10. Matrix of actually available event sources.

## 6. Update rules

- When a requirement changes, update the source document and this matrix in the same PR.
- Every P0 requirement must have a P0 test.
- If a test fails, open a bug linked to the requirement and the test.
- A non-testable requirement must be rewritten or justified.
- Before the release, generate a snapshot of the matrix with commit/version.
- Do not automatically turn `SPECIFIED` into `IMPLEMENTED` or `PASS`.
- Coverage is calculated separately for specification, implementation, and passed tests.

## 7. Progress indicators

- Specification coverage = requirements with a valid document / total requirements.
- Implementation coverage = requirements with a verified implementation / total requirements.
- Test coverage = requirements with at least one executed test / total requirements.
- Pass rate = PASS tests / executed tests (excluding NOT RUN/BLOCKED/SKIPPED).
- Residual risk = P0 requirements without a PASS test + open bugs by severity.

These indicators must not be aggregated into a single number that hides critical gaps.

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

### Traceability update 1.1

Requirements `TR-ARC-006` and `TR-ARC-007` no longer classify Python and Rust as optional languages: they are approved stack components with distinct roles. Their implementation remains tracked as `TO BE IMPLEMENTED` until code and evidence exist. Requirements were added for the dashboard, monorepo/CI, MIT, optional Docker, and DevSecOps controls.
