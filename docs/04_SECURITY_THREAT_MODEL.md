# HSOC — Security Threat Model

**Version:** 1.1-draft | **Method:** asset → threat → control → test

## 1. Purpose and boundaries

Model the threats to the local service, the UI, the database, configurations, backups, notifications, and event sources. The model must be updated when architecture, permissions, IPC, or functionality change.

## 2. Assets

- DB integrity and availability.
- Configurations and policies.
- Telegram credentials and other optional tokens.
- Logs, events, incidents, and inventory.
- Integrity of updates and binaries.
- IPC channel.
- Backups and encryption keys.
- Service identity and authorizations.

## 3. Actors and surfaces

- Authorized local user.
- Unauthorized or compromised local process.
- Malformed OS input/events.
- Vulnerable dependencies.
- Tampered update package.
- Untrusted IoT device.
- External notification channel.

Surfaces: installer, service, IPC, event parsers, configuration, DB, backup, updater, UI, integrations.

## 4. Threats and mitigations

| ID | Threat | Primary mitigation |
|---|---|---|
| TH-01 | Unauthorized local process uses IPC | ACL, identity verification, per-operation authorization |
| TH-02 | UI sends malformed input | Schema validation, limits, fuzz/unit tests |
| TH-03 | Crafted event causes a crash | Robust parser, limits, error isolation |
| TH-04 | Excessive service privileges | Least privilege, explicit capabilities |
| TH-05 | Corrupted or modified DB | File ACL, integrity, verified backups |
| TH-06 | Secrets in logs/config/backups | OS Credential Store, redaction, encryption |
| TH-07 | Tampered update | Package signing and verification |
| TH-08 | Vulnerable dependency | Inventory, advisory scan, controlled update |
| TH-09 | False positive triggers a wrong response | Confidence, rationale, human approval |
| TH-10 | Forged notification used as a command | Notifications informational only, no remote commands |
| TH-11 | Unobservable IoT | Declare limits, do not infer security from missing data |
| TH-12 | Unrestorable backup | Checksums, periodic restore tests |
| TH-13 | Diagnostic export leaks data | Redaction, preview, minimization |
| TH-14 | Compromised optional module | Isolation, narrow interface, no DB access |
| TH-15 | Altered config weakens policy | Validation, audit, security constraints that cannot be disabled |

## 5. Control principles

- Secure defaults and fail-safe.
- Least privilege and UI/service separation.
- No arbitrary commands.
- Untrusted input validated.
- Secrets kept in the operating system.
- Verified updates.
- Audit for sensitive operations.
- Automated responses disabled initially.
- Tested backups.
- No remote port open by default.

## 6. Software security incident handling

1. Detect and classify the problem.
2. Preserve logs and the affected version, without collecting unnecessary data.
3. Limit the vulnerable function with a safe policy.
4. Prepare patch and tests.
5. Independent review.
6. Publish an advisory and signed update when necessary.
7. Verify the fix and update the threat model.

## 7. Security tests

| ID | Test | Expected result |
|---|---|---|
| SEC-01 | IPC client without permission | Access denied |
| SEC-02 | Input with unexpected fields | Rejection or controlled ignoring according to schema |
| SEC-03 | Huge payload | Rejection within limits |
| SEC-04 | Shell injection attempt in fields | No command executed |
| SEC-05 | Search for tokens in logs | No secrets found |
| SEC-06 | Check data directory permissions | Authorized identities only |
| SEC-07 | Config that disables a mandatory security control | Rejected |
| SEC-08 | Unsigned/tampered update package | Installation blocked |
| SEC-09 | Diagnostic export | Secrets and unnecessary data redacted |
| SEC-10 | Optional worker unavailable | Core remains stable and reports degradation |
| SEC-11 | DB error during an operation | No inconsistent partial write |
| SEC-12 | Unauthorized restore | Denied and audited |
| SEC-13 | Notification with hostile content | Displayed as text, never executed |
| SEC-14 | Service started without unnecessary elevated privileges | Core functions operational |
| SEC-15 | Known vulnerable dependencies in CI | Pipeline flags/blocks according to severity |
| SEC-16 | Permission regression test | No undocumented expansion |

## 8. Residual risks

- An already compromised endpoint can forge or suppress events.
- Consumer routers may not expose sufficient telemetry.
- A user with administrative privileges can alter the local system.
- No product can guarantee complete detection.
- The model does not replace a professional audit before public distribution.

## 9. Gate

Before every release: updated threat model, SEC tests executed, known vulnerabilities assessed, signature verified, backup/restore tested, and no open critical risk without formal acceptance.

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

### Additional risks introduced by the multi-language stack

- **Go/Rust/Python boundaries:** validate schema, sizes, encoding, and versions at every boundary; handle timeouts, crashes, and incomplete results.
- **Rust FFI:** keep interfaces small, document ownership and lifetimes, handle panic boundaries, and do not transfer implicit privileges.
- **Python workers:** isolated and restricted execution; no execution of generated code or arbitrary commands; pinned and verified dependencies.
- **Tauri dashboard:** expose only the necessary capabilities, reduce the Tauri commands available to the UI, and do not treat the frontend as an authorization boundary.
- **Docker:** not required in the product; test containers must not receive secrets or non-essential host mounts.
- **V2 remote agents:** will require a dedicated threat model for identity, enrollment, revocation, mutual authentication, replay protection, and updates.
