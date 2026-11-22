# HSOC — User & Administrator Guide

**Version:** 1.1-draft | **Audience:** local users and maintainers

## 1. Introduction

HSOC collects and organizes the security signals available on the computer. The dashboard lets you review events, detections, incidents, inventory, and service status. The product is assistive: it does not guarantee that every threat is identified.

## 2. Startup and status

The UI shows:
- service available/unavailable;
- collector status;
- last event acquired;
- DB status;
- last backup;
- any vulnerability checks in progress;
- HSOC version and protocol version.

If the service is unavailable, the UI must show an understandable message and a diagnostic ID, not stack traces or secrets.

## 3. Dashboard

Recommended sections:
1. Overview.
2. Events.
3. Detections.
4. Incidents.
5. Software inventory.
6. Vulnerabilities.
7. Backup and restore.
8. Configuration.
9. Notifications.
10. Status and diagnostics.

Every detection must show source, date, rule/version, severity, confidence, rationale, and suggested actions.

## 4. Incidents

Suggested states:
- `OPEN`
- `ACKNOWLEDGED`
- `INVESTIGATING`
- `RESOLVED`
- `FALSE_POSITIVE`

Closing as a false positive must preserve the trail and, where possible, the reason. Do not silently delete events.

## 5. Configuration

- Settings must explain the effect of each option.
- Risky options off by default.
- Security policies that cannot be disabled must not appear as ordinary toggles.
- Invalid config must not replace the last valid configuration.
- Important changes are audited.

## 6. Telegram and notifications

Telegram is optional and disabled initially. Enabling it must:
- be explicit;
- explain which data is sent and to which service;
- store the token in the operating system Credential Store;
- not send unnecessary sensitive payloads;
- allow testing and revocation;
- not accept remote commands in response to notifications.

## 7. IoT

HSOC can only show information obtainable passively or through supported sources. Some routers do not expose enough data. A device not detected does not mean a secure device; a device detected does not mean a compromised device.

## 8. Backup and restore

- Create a backup before important updates.
- Verify checksum and backup status.
- Periodically perform a test restore in a separate environment.
- Before restoring, show which DB will be replaced and which data may be lost.
- Do not overwrite the active DB without confirmation.

## 9. Diagnostics

Diagnostic export:
- must be voluntary;
- must show a preview of the contents;
- must redact tokens, credentials, and unnecessary data;
- must include versions, health, errors, and minimal technical information;
- must not automatically send the file to third parties.

## 10. Troubleshooting

| Symptom | Check |
|---|---|
| UI cannot connect | Verify service and IPC ACL |
| Service does not start | Consult redacted logs and health |
| No events | Verify collector status and OS limits |
| Too many alerts | Review rule, version, confidence, and fixtures |
| Backup failed | Space, permissions, WAL, and checksum |
| Restore unavailable | Verify DB version and backup integrity |
| Telegram does not send | Opt-in status, token, and connectivity |

## 11. Privacy

Data is local by default. Before enabling external integrations, explain the shared data. The user must be able to disable Telegram and remove credentials. Retention and export must be transparent.

## 12. Usability tests and guides

- UI-01: overview shows correct health.
- UI-02: event and incident are distinguishable.
- UI-03: severity and confidence have accessible definitions.
- UI-04: an IPC error does not cause a frozen screen.
- UI-05: optional settings off on first start.
- UI-06: restore shows confirmation and consequences.
- UI-07: export shows preview/redaction.
- UI-08: IoT limits are explicit.
- UI-09: the installation guide matches the actual installer.
- UI-10: documentation updated per release.

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

### Note for the end user

The user must not install Docker or development toolchains (Go, Rust, Python, Node.js). The HSOC installer will distribute the necessary components. The dashboard is a Tauri desktop application with a React/TypeScript interface; the Go service can keep running even when the dashboard is closed. The internal Rust/Python modules require no separate configuration by the user.
