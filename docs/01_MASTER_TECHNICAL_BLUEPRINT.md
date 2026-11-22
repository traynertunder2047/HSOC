# HSOC — Master Technical Blueprint

**Version:** 1.1-draft | **Status:** Project Specification

## 1. Purpose

HSOC (Home Security Operations Center) is a local-first desktop application designed to help users monitor the security status of their Windows computer and, within the limits of available data, certain aspects of IoT devices connected to their home network.

It is not a video surveillance product, does not require cameras or dedicated sensors, and must not require external hardware.

Objectives: local event collection, software inventory, rule-based detection, incident management, dashboard, auditing, backup and recovery, and optional notifications.

## 2. Non-Negotiable Principles

1. Accessibility and minimal operating costs; no paid cloud service is required for basic operation.
2. Local-first: primary processing and storage take place on the device.
3. No mandatory HSOC login in V1; the operating system's identity, session, and permissions are reused.
4. Least privilege: the service operates with the lowest privileges compatible with the required functions.
5. Secure by default: external telemetry, Telegram, and automated responses are disabled by default.
6. Transparency: every detection displays its rationale, source, confidence level, and rule version.
7. Reversibility: backups, restore, rollback, and reversible actions wherever possible.
8. No guarantee of complete detection or zero false positives.
9. V1 consists of a single local agent; remote agents and distributed coordination are V2+.
10. Passive and non-intrusive IoT monitoring; coverage depends on the router and available information.

## 3. Functional Scope

### V1-MVP (P0)

- Local HSOC service written in Go.
- Startup, controlled shutdown, and health status.
- Collection from one supported Windows event source.
- Event validation and normalization.
- SQLite persistence.
- A non-destructive demonstration detection rule.
- Creation and viewing of an incident.
- Desktop dashboard displaying events, detections, incidents, and service status.
- Technical logs and essential diagnostics.

### V1 (P0/P1)

- Software and component/dependency inventory.
- Versioned and configurable rules with defined limits.
- Basic deduplication and correlation.
- Verified backup and recovery.
- Configuration and policy management.
- Local notifications; Telegram only as an opt-in integration.
- System and dependency vulnerability checks, performed monthly and outside the regular cycle for critical issues.
- Simulation mode for testing rules and policies without modifying the system.
- Auditing of sensitive operations.

### V2+

- Remote agents with identity, enrollment, revocation, and an authenticated protocol.
- Linux and macOS support after Windows has been stabilized.
- Multi-node coordination and distributed policies.
- More advanced response automation, only after risk analysis and explicit approval.

## 4. Out of Scope

- Video surveillance, cameras, microphones, or dedicated sensors.
- Any requirement to purchase hardware.
- Mandatory central cloud infrastructure.
- Remote access exposed to the Internet.
- Intrusive or aggressive IoT device scans.
- Execution of arbitrary commands received through notifications or external interfaces.
- Automatic modification of production code or unreviewed automatic fixes.
- A guarantee of complete protection against every threat.

## 5. Logical Architecture

```text
+-----------------------------+
| Desktop UI                  |
| Tauri + React + TypeScript  |
+--------------+--------------+
               | Local IPC
+--------------v--------------+
| HSOC Core Service (Go)      |
| API/IPC, policy, orchestration |
| collection, detection, audit |
+------+-----------+----------+
       |           |
       |           +--------------------+
       v                                v
+--------------+                +---------------+
| SQLite       |                | OS adapters   |
| local state  |                | Windows first |
+--------------+                +---------------+
       |
       +---- backup/restore
       +---- optional notification adapter

Optional isolated workers:
- Rust: specialist security modules, with clearly defined interfaces and responsibilities.
- Python: workers and auxiliary analysis tasks, with validated input/output.
- Go remains the core; Rust and Python do not access SQLite directly.