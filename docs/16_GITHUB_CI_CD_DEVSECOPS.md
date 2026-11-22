# Document 16 – GitHub, CI/CD, and DevSecOps

**Project:** HSOC – Home Security Operations Center  
**Document version:** 1.1 – updated design draft  
**Status:** Proposed specification, to be approved and implemented  
**Date:** November 15, 2026

---

## 1. Purpose

This document defines how to organize the HSOC GitHub repository and how to automate quality checks, tests, security, and release preparation.

The objectives are:

- identify errors before changes reach the main branch;
- automatically verify compilation and tests;
- check dependencies and identify known vulnerabilities;
- reduce the risk of publishing secrets or personal data;
- make builds and verifications reproducible;
- keep development, testing, and distribution separate.

CI does not certify that the product is secure: it only verifies the automated checks that have been configured.

## 2. Principles and constraints

1. **Local-first:** Docker and cloud services must not be required to run HSOC on the user's computer.
2. **Zero initial cost:** use the available free features, respecting their limits and conditions. Do not depend on paid services.
3. **Windows first:** the first complete pipeline must verify Windows. Linux and macOS will be added when support is implemented.
4. **Least privilege:** workflows and tokens must have only the strictly necessary permissions.
5. **Fail closed for essential checks:** a failed mandatory check blocks the merge.
6. **Reproducibility:** tool versions and dependencies must be declared and kept under control.
7. **No automated action on the user's system:** CI tests the software; it does not execute threat responses on user devices.
8. **No secrets in the repository:** Telegram tokens, keys, credentials, and local data must not be committed.

## 3. Repository and structure

Recommended initial structure:

```text
HSOC/
├── .github/
│   ├── workflows/
│   │   ├── ci-go.yml
│   │   ├── ci-dashboard.yml
│   │   ├── ci-rust.yml
│   │   ├── ci-python.yml
│   │   ├── security.yml
│   │   └── release.yml
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
├── cmd/                     # Go entry points
├── internal/                # Go core and modules
├── rust/                    # specialist security modules
├── python/                  # workers and auxiliary analysis
├── desktop/
│   ├── src/                 # React + TypeScript
│   └── src-tauri/           # Tauri shell
├── database/                # schema, migrations, queries
├── protocols/               # IPC, events, messages, and schemas
├── rules/
├── configs/
├── installers/windows/
├── docs/
├── tests/
├── docker/                  # optional: development/testing
├── scripts/
├── .gitignore
├── README.md
├── SECURITY.md
├── CONTRIBUTING.md
├── LICENSE
└── CHANGELOG.md
```

Go, Rust, and Python are part of the approved stack; their modules are integrated in phases. There is no need to create empty directories or modules before starting the related implementation. The actual structure must stay aligned with the architecture approved in the previous documents.

### 3.1 Branches

- `main`: stable branch; changes only through Pull Requests.
- `feature/<description>`: new functionality.
- `fix/<description>`: fixes.
- `docs/<description>`: documentation.
- `security/<description>`: security fixes, with appropriate access and review.

For an individual project, review can be a documented personal checklist; an independent review must not be faked if it did not happen.

### 3.2 Protection of `main`

Configure, where available:

- mandatory Pull Request;
- mandatory CI checks;
- blocking of force-pushes;
- blocking of branch deletion;
- resolution of conversations before merge;
- at least one independent review once the project has more collaborators.

Do not enable rules that require a number of approvals impossible to satisfy for a repository managed by a single person.

## 4. CI strategy

CI (Continuous Integration) runs on Pull Requests and on updates to the relevant branches. A scheduled run is also planned for periodic checks.

### 4.1 Go pipeline – main agent

Planned checks:

- formatting with `gofmt`;
- analysis with `go vet`;
- tests with `go test ./...`;
- compilation for the supported Windows target;
- Go module vulnerability check with `govulncheck`;
- verification that `go.mod` and `go.sum` are consistent.

Indicative commands:

```bash
gofmt -w .
go vet ./...
go test ./...
govulncheck ./...
go build ./...
```

In CI, the formatting check must verify the repository without modifying files and fail if it finds unformatted files. The `gofmt -w` command is useful locally, not as a silent commit fix in the workflow.

### 4.2 Rust pipeline – specialist modules

To be enabled when Rust code is introduced:

- `cargo fmt --check`;
- `cargo clippy -- -D warnings`;
- `cargo test`;
- dependency check with `cargo audit`;
- build for the targets actually supported.

### 4.3 Python pipeline – auxiliary modules

To be enabled when Python is introduced:

- lint and formatting with a chosen and versioned tool, for example Ruff;
- tests with `pytest`;
- dependency check with `pip-audit`;
- build or packaging only if required by the component.

Dependencies must be pinned or locked with a lockfile, according to the packaging system adopted.

### 4.4 Dashboard – React and TypeScript

To be enabled when the dashboard is introduced:

- reproducible dependency installation through a lockfile;
- lint;
- TypeScript type check;
- automated tests;
- production build.

Use `npm ci` if the project adopts npm and maintains `package-lock.json`; for other package managers, use the corresponding deterministic command.

### 4.5 Integration tests

Integration tests verify the boundaries between components, in particular:

1. acquisition and validation of a test event;
2. normalization by the Go agent;
3. persistence in the SQLite database;
4. generation of a demonstrative event or incident;
5. display in the dashboard, when available;
6. behavior when the database is unavailable or data is invalid.

Tests must use synthetic data and temporary environments. They must not require access to the user's real devices.

## 5. Supply chain and repository security

### 5.1 Secrets

- Enable secret scanning and push protection, where available.
- Do not place tokens, passwords, private keys, `.env` files, or backups in the repository.
- Use GitHub Secrets only for credentials strictly necessary for workflows.
- Do not print secrets in logs.
- Immediately revoke and replace an accidentally published secret: deleting it from the latest commit is not enough.

### 5.2 Dependencies

- Prefer maintained and necessary dependencies.
- Pin versions and update in a controlled way.
- Assess provenance, license, maintenance, and known vulnerabilities.
- Use Dependabot or equivalent tools when compatible with the plan and the repository.
- Automated update pull requests must pass tests and checks before merging.

### 5.3 Workflow permissions

Every workflow must explicitly declare the minimal `GITHUB_TOKEN` permissions. For test pipelines, start from read-only permissions, for example `contents: read`.

Do not use `write-all`. Do not run untrusted Pull Request code with secrets available. For Pull Requests from forks, keep workflows without write privileges and without secrets.

### 5.4 GitHub Actions

- Prefer official or widely verified actions.
- Pin actions to a stable version and, for sensitive workflows, to the full commit SHA.
- Periodically update the pinned versions.
- Do not run unverified remote scripts.

## 6. Security scans

The security pipeline may include:

| Check | Purpose |
|---|---|
| Secret scanning | Identify accidentally committed credentials |
| Dependency scanning | Detect known vulnerabilities in dependencies |
| Static analysis | Identify classes of errors in the code |
| License review | Identify incompatible licenses or licenses needing assessment |
| SBOM | Produce an inventory of software components |
| Container scanning | Check Docker images, if used |

Scanning must not be interpreted as a guarantee that no vulnerabilities exist. Every result must be classified by severity, impact, and applicability to the project.

## 7. Docker: optional use

Docker is a tool for development and testing, not a requirement for installing or running HSOC.

Allowed and useful uses:

- reproducible auxiliary environments;
- isolated integration tests;
- running analysis tools;
- temporary test services, if needed in the future.

Constraints:

- the HSOC desktop agent must not require Docker;
- do not run the agent with elevated privileges inside a container to bypass container limits;
- do not mount personal directories or host sockets without a documented need;
- do not put credentials in images;
- use reliable, versioned, updated base images;
- generate and check the SBOM of images, if published.

A `Dockerfile` or `compose.yaml` should be added only when a concrete use case exists. Avoid a Compose file made of fictitious or unnecessary services.

## 8. CD and releases

Continuous Delivery (CD) prepares the artifacts; publication must be a deliberate and verifiable step.

### 8.1 Conditions for a release

A release cannot be published as stable if:

- mandatory tests fail;
- the build is not reproducible or has not been verified;
- there are blocking vulnerabilities that have not been assessed;
- release notes are missing;
- the installation and uninstallation path has not been verified;
- backup and restore have not been tested for changes involving data.

### 8.2 Artifacts

Based on the actual state of the project, a release may contain:

- Windows installer;
- artifact checksums;
- SBOM;
- release notes;
- update and restore instructions.

Digital signing of installers is recommended before public distribution. It may involve costs or external requirements: it must not be promised as a guaranteed free feature.

### 8.3 Versioning

Adopt Semantic Versioning when the API and the product are sufficiently stable:

- `MAJOR`: incompatible changes;
- `MINOR`: compatible functionality;
- `PATCH`: compatible fixes.

During initial development, use `0.x` versions and clearly state that the software is experimental.

## 9. Repository documentation

Minimum files:

- `README.md`: purpose, status, requirements, starting, and structure;
- `SECURITY.md`: how to report a vulnerability responsibly;
- `CONTRIBUTING.md`: conventions, tests, and Pull Requests;
- `CHANGELOG.md`: changes per version;
- `LICENSE`: consciously chosen license;
- Issue and Pull Request templates.

Do not choose a license by assumption: the decision must be explicit and consistent with the dependencies and the distribution model.

## 10. Acceptance criteria

The CI/CD configuration is acceptable when:

1. a Pull Request runs the planned checks for the components present;
2. an error in a mandatory check prevents merging into `main`;
3. workflows use minimal permissions;
4. secrets are not exposed in logs or in untrusted workflows;
5. dependencies are checked;
6. tests use synthetic data and do not modify the host system;
7. Docker is not required to install or start HSOC;
8. a release includes verifiable artifacts and information;
9. scans and results are retained in a consultable form;
10. every check not yet implemented is declared as such.

## 11. Implementation sequence

| Phase | Activity | Priority |
|---|---|---|
| 1 | Create the repository and the minimal structure | P0 |
| 2 | Add README, SECURITY, CONTRIBUTING, LICENSE to be decided, and CHANGELOG | P0 |
| 3 | Configure Go CI for formatting, analysis, tests, and build | P0 |
| 4 | Protect `main` and make CI checks mandatory | P0 |
| 5 | Add secret and dependency scanning | P0 |
| 6 | Add integration tests and backup/restore tests | P0 |
| 7 | Add dashboard, Rust, and Python CI when the modules exist | P1 |
| 8 | Introduce Docker only for documented development/test use cases | P1 |
| 9 | Prepare the release workflow and SBOM | P1 |
| 10 | Evaluate installer signing and the Linux/macOS matrix | P2 |

## 12. Decisions still to be confirmed

| ID | Decision | Status |
|---|---|---|
| ADR-CI-01 | GitHub Actions as the CI platform | Project choice to be configured |
| ADR-CI-02 | `main` protected with Pull Requests and mandatory checks | Proposal |
| ADR-CI-03 | Windows as the first complete CI target | Consistent with the project direction |
| ADR-CI-04 | Docker optional, not required of the end user | Confirmed |
| ADR-CI-05 | Scanning tools and blocking thresholds | To be defined during implementation |
| ADR-CI-06 | Project license | To be decided |
| ADR-CI-07 | Installer signing and distribution strategy | To be decided |

## 13. Distinction between specified, implemented, and verified

This document describes a proposed configuration. The presence of a workflow in this specification does not mean the workflow has been created, executed, or passed.

For each component, keep three distinct states:

- **SPECIFIED:** behavior and checks described;
- **IMPLEMENTED:** files, code, and configuration present in the repository;
- **VERIFIED:** checks executed and results recorded.

Do not declare a release, a scan, or a test as passed without a verifiable result.

---

**End of Document 16 – GitHub, CI/CD, and DevSecOps**


## 14. HSOC decisions consolidated in revision 1.1

- Project license: MIT (`LICENSE` at the repository root).
- Repository: GitHub monorepo.
- Dashboard: Tauri + React + TypeScript.
- Approved languages: Go, Rust, and Python. Go is the core and sole SQLite writer; Rust is intended for specialist security components; Python for workers and auxiliary analysis.
- CI includes distinct workflows for Go, Rust, Python, and the dashboard, applied to the components actually present.
- Docker is optional and limited to development/testing; it is not required of the end user.
- Remote agents: functional implementation from V2; V1 excludes remote listeners and agents.
- Windows x86-64 is the first target; Linux/macOS will be added after stabilization.

The decision to use three languages does not imply three mandatory processes for every function. Every component must have defined responsibilities, contract, tests, and packaging plan. The documentation does not prove that the workflows are already configured in the repository.
