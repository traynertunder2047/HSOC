# HSOC — API & IPC Specification

**Version:** 1.1-draft | **Status:** contracts to be consolidated

## 1. Objective

Define communication between the desktop dashboard and the local Go service, without direct database access. IPC is a security boundary: UI input is always untrusted.

## 2. Architecture

- V1 Windows: evaluate local Named Pipes.
- Restrictive ACLs: only authorized accounts/sessions.
- The service verifies the client's identity/permissions and does not trust the pipe name alone.
- No TCP listener exposed by default.
- The UI can connect/disconnect without stopping the service.
- The service can operate without the UI.
- Protocol with an explicit version and controlled compatibility.

## 3. Operation families

| Family | Operations |
|---|---|
| Health | `GetHealth`, `GetVersion`, `GetCapabilities` |
| Events | `ListEvents`, `GetEvent` |
| Detection | `ListDetections`, `GetDetection` |
| Incidents | `ListIncidents`, `GetIncident`, `UpdateIncidentStatus` |
| Inventory | `ListInventory`, `GetInventoryItem` |
| Vulnerability | `ListFindings`, `GetScanStatus`, `RequestScan` |
| Configuration | `GetConfig`, `ValidateConfig`, `ApplyConfig` |
| Rules | `ListRules`, `SetRuleEnabled` |
| Backup | `CreateBackup`, `ListBackups`, `VerifyBackup`, `RestoreBackup` |
| Diagnostics | `GetDiagnostics`, `ExportDiagnostics` |
| Simulation | `RunSimulation`, `GetSimulationResult` |
| Notifications | `GetNotificationStatus`, `TestNotification` |

Destructive or sensitive operations require authorization, audit, and, where specified, explicit approval.

## 4. Message contract

Recommended logical format:
```json
{
  "protocol_version": "1",
  "request_id": "unique-id",
  "operation": "GetHealth",
  "payload": {},
  "client_metadata": {
    "ui_version": "x.y.z"
  }
}
```

Response:
```json
{
  "protocol_version": "1",
  "request_id": "same-id",
  "ok": true,
  "data": {},
  "error": null
}
```

Names and types are illustrative: the formal schema must be versioned and validated. Do not include secrets in metadata.

## 5. Error model

Logical codes:
- `INVALID_ARGUMENT`
- `UNAUTHENTICATED`
- `PERMISSION_DENIED`
- `NOT_FOUND`
- `CONFLICT`
- `FAILED_PRECONDITION`
- `RESOURCE_EXHAUSTED`
- `UNAVAILABLE`
- `INTERNAL`
- `UNSUPPORTED_VERSION`

Errors must not expose stack traces, sensitive paths, tokens, or internal details to the UI. Technical logs may contain correlation IDs and redacted context.

## 6. IPC rules

1. Validate the maximum message size.
2. Validate schema, enums, ranges, and encoding.
3. Apply timeouts and cancellation.
4. Limit concurrent requests and rates.
5. Reject unsupported versions.
6. Do not execute shell commands from IPC input.
7. Authorize every sensitive operation on the Go side.
8. Audit configuration changes, policies, restores, and actions.
9. No plaintext secrets in logs.
10. Close the channel cleanly and recover from a restarted UI.

## 7. Authorization

- No dedicated HSOC login in V1.
- Use the Windows identity and the local channel ACL.
- The UI is not automatically trusted: the service validates every request.
- For operations requiring elevation, prefer a scoped and documented helper/operation; do not make the entire service an administrator.
- Explicitly confirm restores, policy changes, and actions with impact.

## 8. Push events

Possible notifications from the service:
- `health.changed`
- `event.received`
- `detection.created`
- `incident.updated`
- `scan.completed`
- `backup.completed`
- `configuration.changed`

Push events are informational; the UI must be able to retrieve state through queries if it misses a notification.

## 9. Versioning

- `protocol_version` is mandatory.
- Compatible additions preferred; removals/different semantics require a new version.
- UI and service negotiate capabilities.
- Compatibility tests between UI N and service N-1 according to the release matrix.
- Contracts and types generated or verified from a shared schema.

## 10. IPC tests

| ID | Test | Expected result |
|---|---|---|
| IPC-001 | Authorized UI connects | Connection established |
| IPC-002 | Unauthorized client | Connection/request denied |
| IPC-003 | Malformed payload | Controlled rejection |
| IPC-004 | Payload over the limit | Rejected without excessive consumption |
| IPC-005 | Service timeout | UI shows an unavailable state |
| IPC-006 | Unknown protocol version | `UNSUPPORTED_VERSION` |
| IPC-007 | Duplicate request ID | Documented deterministic behavior |
| IPC-008 | Unauthorized restore request | Denied and audited |
| IPC-009 | UI closed during push | Service continues, UI recovers state |
| IPC-010 | Input containing commands | Treated as data, never executed |
| IPC-011 | Request flood | Rate/concurrency limit applied |
| IPC-012 | Secrets in an error | No disclosure in messages/logs |

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

### Compatibility with the approved stack

The V1 desktop IPC connects the Tauri/React/TypeScript dashboard to the Go service. The specialist Rust modules and Python workers communicate with Go through narrow, versioned internal interfaces; they must not be treated as privileged clients. Every message coming from a separate component is validated as untrusted input.

The schemas in `protocols/` must distinguish:
- V1 dashboard-service local IPC;
- internal Go-Rust and Go-Python contracts;
- event and data schemas;
- future V2 remote agent contracts, disabled and not exposed in V1.
