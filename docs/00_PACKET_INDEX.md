# HSOC — Documentation Package Index

**Documentation version:** 1.1-draft  
**Date:** 2026-11-15  
**Status:** Consolidated technical draft — does not imply that the software has been implemented or verified.

## Contents

| File | Contents |
|---|---|
| `01_MASTER_TECHNICAL_BLUEPRINT.md` | Vision, requirements, architecture, and system boundaries |
| `02_DATABASE_SPECIFICATION.md` | SQLite data model, integrity, migrations, retention, and backups |
| `03_API_IPC_SPECIFICATION.md` | Internal contracts, IPC, events, errors, and versioning |
| `04_SECURITY_THREAT_MODEL.md` | Assets, threats, trust boundaries, mitigations, and security tests |
| `05_DEPLOYMENT_INSTALLATION.md` | Windows installation, service, dashboard, updates, and uninstallation |
| `06_IMPLEMENTATION_BLUEPRINT.md` | Modules, repository structure, technology stack, and development sequence |
| `07_TESTING_QA.md` | QA strategy, test cases, metrics, and release criteria |
| `08_USER_ADMIN_GUIDE.md` | Usage, configuration, privacy, troubleshooting, and recovery |
| `09_RELEASE_MAINTENANCE_PLAN.md` | Versions, maintenance, vulnerabilities, SBOM, and releases |
| `10_DETECTION_RESPONSE_SPECIFICATION.md` | Data collection, detection, incidents, response, and false positives |
| `11_CONFIGURATION_POLICY_SPECIFICATION.md` | Configuration, policies, secure defaults, and change management |
| `12_IMPLEMENTATION_READINESS.md` | Consolidation, Definition of Ready/Done, vertical slice, and gates |
| `13_DEVELOPMENT_RULES.md` | Mandatory rules during design and development |
| `14_TRACEABILITY_MATRIX.md` | Requirement-document-module-test-version traceability matrix |
| `15_HSOC_MASTER_SUMMARY.md` | Complete overview and integrated roadmap |
| `16_GITHUB_CI_CD_DEVSECOPS.md` | Repository, GitHub Actions, security checks, optional Docker, and releases |

## Status Conventions

- **SPECIFIED:** A requirement or behavior is described in the documentation.
- **TO BE CONFIRMED:** A technical decision has not yet been definitively approved.
- **TO BE IMPLEMENTED:** The existence of the corresponding code must not be assumed.
- **TO BE TESTED:** The test has been defined, but no execution evidence has been provided.
- **SUPERSEDED:** A decision or requirement has been replaced by a later revision.

## Priority and Version Conventions

- **P0:** Required for a functional and secure MVP.
- **P1:** Important for the first stable release.
- **P2:** Optional improvement for V1.
- **P3:** Future functionality, normally V2 or later.
- **V1-MVP:** First demonstrable vertical slice.
- **V1:** First stable release.
- **V2+:** Subsequent development.

## Fundamental Rule

This package is a project baseline. No item marked as specified proves that it has been implemented; no defined test proves that it has passed. Every implementation and every test result must be accompanied by a commit/version, log, or reproducible report.

## Decisions Consolidated in Revision 1.1

- MIT License.
- GitHub monorepo.
- Tauri + React + TypeScript dashboard.
- Go + Rust + Python technology stack: Go core/SQLite, Rust specialist security modules, Python workers and analysis.
- Docker is optional and is not required by end users.
- Remote agents will be implemented starting with V2; no remote functionality will be active in V1.
- Document 16 is included in the package.
- Repository support files are included in the ZIP package: `LICENSE`, `README.md`, `.gitignore`, `SECURITY.md`, `CONTRIBUTING.md`, and others.

Revision 1.1 updates the documentation baseline. It does not certify that any code or tests have been implemented.