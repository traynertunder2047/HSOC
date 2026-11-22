# HSOC — Database Specification

**Version:** 1.1-draft | **Database:** local SQLite

## 1. Principles

- SQLite is the primary local database for V1.
- Only the Go service may write to or migrate the database.
- The UI, Python/Rust workers, and future processes use the API/IPC.
- Foreign keys enabled; transactions for multi-table operations.
- Versioned, atomic, and tested migrations.
- Secrets are not stored in plaintext in the DB: use Credential Manager/OS secret store.
- Timestamps in UTC, local display in the UI.
- WAL may be adopted if compatible with the backup plan; the backup must be WAL-aware.

## 2. Logical entities

| Entity | Purpose | Indicative fields |
|---|---|---|
| `schema_migrations` | Schema version | version, applied_at, checksum |
| `events` | Normalized events | id, source, event_type, occurred_at, received_at, payload, fingerprint |
| `detections` | Rule results | id, event_id, rule_id, rule_version, severity, confidence, rationale |
| `incidents` | Aggregated case | id, title, status, severity, opened_at, updated_at |
| `incident_events` | Many-to-many link | incident_id, event_id |
| `rules` | Rule metadata | rule_id, version, enabled, definition, updated_at |
| `inventory_items` | Software and components | id, name, version, publisher, source, observed_at |
| `vulnerability_findings` | Check results | id, component_id, advisory_id, severity, status, detected_at |
| `actions` | Proposed/executed actions | id, incident_id, action_type, status, created_at |
| `approvals` | Approvals | id, action_id, decision, decided_at, actor |
| `audit_log` | Operations trail | id, actor, operation, target, result, timestamp |
| `config_snapshots` | Configuration versions | id, version, content_hash, created_at |
| `backup_records` | Backup inventory | id, path, created_at, checksum, status |
| `health_snapshots` | Component status | id, component, status, detail, observed_at |

The final physical schema (types, indexes, CHECK constraints, FKs, nullability, and migrations) must be fixed in an ADR/schema review before any persistent code.

## 3. Constraints and indexes

- Stable primary keys; prefer UUID/ULID or service-generated IDs.
- `events.fingerprint` indexed for dedup.
- Indexes on timestamps, severity, status, and FKs used by UI queries.
- CHECK constraints for enums and numeric ranges.
- FKs with `RESTRICT`/`CASCADE` behavior decided per relationship; do not use destructive cascades without justification.
- `incident_events` with composite unique `(incident_id, event_id)`.
- Rules and findings keep version and provenance.
- Payloads with a maximum size and an explicit schema.

## 4. Deduplication

Dedup must not delete the original evidence. Define:
1. deterministic fingerprint over normalized fields;
2. configurable time window;
3. handling of null fields;
4. behavior on collision;
5. link between repeated occurrences and the aggregate record.

**Open decision:** UNIQUE constraints with nullable columns can have unintuitive semantics in SQLite. Consider a non-null fingerprint computed by the service or a partial unique index.

## 5. Integrity and concurrency

- `PRAGMA foreign_keys=ON` for every connection.
- Explicit transactions for event → detection → incident.
- Limited retries for `SQLITE_BUSY`; no infinite loops.
- Defined timeouts and connection pool.
- `integrity_check` and `foreign_key_check` in diagnostics/maintenance.
- Treat disk full, read-only file, wrong permissions, and corruption as explicit errors.
- The UI must not open the DB file or copy it while it is active.

## 6. Retention and privacy

- Configurable retention with a documented conservative default.
- Batch deletion and audit of the operation.
- Separate retention for events, incidents, audit, and backups.
- Do not record tokens, passwords, keys, or unnecessary personal data.
- Diagnostic export with secret redaction and data minimization.

## 7. Backup and restore

- Backup performed by the service using a consistent SQLite API or a procedure that correctly includes WAL.
- Manifest with schema version, UTC timestamp, checksum, and HSOC version.
- Backup encrypted when it contains sensitive data, with the key managed outside the backup.
- Checksum verification before restore.
- Restore preferably to a temporary file; integrity verification; atomic replacement when possible.
- Keep at least one previous copy until the restore has been verified.
- Do not overwrite the active DB without confirmation and a recovery procedure.

## 8. Migrations

Every migration must:
- have an ID and checksum;
- be transactional when supported;
- be tested from N-1 to N;
- handle a pre-migration backup;
- be idempotent or clearly prevent double execution;
- not lose data silently;
- have a rollback plan or state why rollback requires a restore.

## 9. Open questions

- `policy_assignments` structure.
- Circular relationship between `approvals` and `actions`.
- Re-enrollment strategy for future remote agents.
- Final keys and indexes for dedup.
- Default retention and space limits.
- DB/backup encryption and compatibility with zero cost.

## 10. Database tests

| ID | Concise procedure | Expected result |
|---|---|---|
| DB-001 | Open a new DB | Schema created at the expected version |
| DB-002 | Run migration N-1 → N | Migration applied once |
| DB-003 | Repeat an already applied migration | No double modification |
| DB-004 | Insert a nonexistent FK | Operation rejected |
| DB-005 | Insert a duplicate event | Dedup according to the documented rule |
| DB-006 | Simulate an error mid-transaction | Full rollback |
| DB-007 | Read-only DB | Error handled, service does not corrupt data |
| DB-008 | Simulated disk full | Visible error, integrity verified |
| DB-009 | Backup with WAL active | Consistent and verifiable backup |
| DB-010 | Restore onto a test DB | Integrity and counts consistent |
| DB-011 | Expired retention | Only eligible records removed |
| DB-012 | Retention audit | Operation recorded without secrets |
| DB-013 | `integrity_check` | OK result on a valid DB |
| DB-014 | Read/write concurrency | No loss or corruption |

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

### Database boundaries with respect to the stack

- The Go service is the only SQLite writer and manages migrations, transactions, backup, and restore.
- Rust modules and Python workers receive the necessary data through controlled internal interfaces and return structured results; they do not open the SQLite file.
- The Tauri/React/TypeScript dashboard uses only documented IPC to the Go service.
- Future V2 remote agents do not receive direct access to the local database: they will use authenticated APIs/protocols to be designed in the V2 phase.
