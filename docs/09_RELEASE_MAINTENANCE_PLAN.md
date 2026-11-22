# HSOC — Release & Maintenance Plan

**Version:** 1.1-draft

## 1. Versioning

Adopt Semantic Versioning:
- MAJOR: API/schema/policy incompatibility.
- MINOR: compatible functionality.
- PATCH: compatible fixes.

Every release must state the service version, UI version, IPC protocol version, DB schema version, and rule set.

## 2. Channels

- Development: internal builds and tests.
- Preview: limited distribution for verification.
- Stable: only after complete gates.

Do not distribute unsigned builds as stable releases.

## 3. Release pipeline

1. Feature freeze.
2. Verify open issues and risks.
3. Full tests.
4. Dependency and secret scans.
5. SBOM.
6. Build and sign.
7. Installation/update tests.
8. Backup/restore tests.
9. Release notes and known issues.
10. Publication and monitoring.

## 4. Vulnerability maintenance

- Monthly check of the OS and the dependencies in use.
- Off-cycle check for critical advisories or widely relevant exploits.
- Record advisory, component, version, impact, mitigation, and deadline.
- Update in a test environment before release.
- Do not apply auto-fixes to production code without human review.
- Auto-fix allowed only as a proposed patch, with tests and review.

## 5. Dependencies and SBOM

- Inventory of direct/transitive dependencies.
- Versioned lockfiles.
- Removal of unused dependencies.
- Verified licenses.
- SBOM for every stable release.
- Scanners in CI and retained reports.

## 6. Support and bug triage

Classify:
- Critical: data loss, unauthorized execution, compromise, or unsafe install/update.
- High: core feature unavailable or significant risk.
- Medium: degradation with a workaround.
- Low: cosmetic or minor defect.

Every bug must state version, environment, steps, expected/actual, redacted logs, and linked requirement and test.

## 7. Compatibility

Maintain a tested matrix with:
- Windows version;
- architecture;
- Go runtime/build version;
- UI/Tauri version;
- DB schema version;
- protocol version;
- upgrade mode.

Do not claim support just because the software appears to start.

## 8. Rollback and migrations

- Backup before migrations.
- Verify restore on the release candidate.
- Documented binary rollback and DB strategy.
- If schema rollback is not safe, require a backup restore.
- Do not automatically delete the pre-update backup.

## 9. Maintenance tests

- MNT-01: monthly dependency scan.
- MNT-02: off-cycle critical advisory.
- MNT-03: SBOM generated and archived.
- MNT-04: artifact signature verified.
- MNT-05: upgrade from the previous release.
- MNT-06: rollback/recovery.
- MNT-07: non-destructive migration.
- MNT-08: changelog and known issues.
- MNT-09: updated ADRs.
- MNT-10: auto-fix patch submitted to review and tests.

## 10. End-of-support criteria

Announce the end of support for a version in advance. Do not force destructive updates. Provide backup, upgrade, and migration instructions.

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

### Component and license maintenance

The periodic scan must cover separately:
- Go modules (`go.mod`/`go.sum`);
- Rust crates (`Cargo.toml` and lockfile);
- Python dependencies and their lockfile;
- Node/Tauri frontend dependencies and lockfile;
- Docker images only if used in workflows or published.

For every release, generate an SBOM inventory of the distributed components and verify license compatibility. The HSOC MIT license does not change or replace third-party component licenses. Keep notices and attributions in `THIRD_PARTY_NOTICES.md` when required.
