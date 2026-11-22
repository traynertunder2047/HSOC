# HSOC — Configuration & Policy Specification

**Version:** 1.1-draft

## 1. Purpose

Separate operational preferences, technical configuration, runtime state, and secrets. A single source of truth must be defined for each data category.

## 2. Categories

- **Config file:** static settings and bootstrap, versioned schema.
- **Database:** runtime state, events, incidents, audit, inventory, backup status.
- **OS Credential Store:** tokens and secrets.
- **Policy:** security limits and authorizations.
- **Environment/build:** non-secret values needed for packaging/testing.

Do not duplicate the same setting across multiple sources without a precedence rule.

## 3. Proposed precedence

1. Built-in, non-disableable security constraints.
2. Documented local administrative policies.
3. Valid local configuration.
4. Product defaults.

Final order to be confirmed with ADR-125. No environment variable or CLI argument may silently disable security controls.

## 4. Schema

Every field must have:
- name and type;
- default;
- range;
- allowed value;
- description;
- risk;
- schema version;
- reload effect;
- any required restart.

## 5. Secure defaults

- Telegram: OFF.
- Automated response: OFF.
- Remote agents: OFF/not present in V1.
- Network listener: absent by default.
- External export: OFF.
- Optional modules: OFF when not installed.
- Logs: no secrets.
- Privileges: minimal.

## 6. Changes

- Validate entirely before applying.
- Apply atomically.
- If invalid, keep the last valid configuration.
- Audit sensitive changes.
- Show diff and impact.
- Support rollback.
- Do not change response policies without confirmation.

## 7. Feature flags

Feature flags are not a security control. They must not be used to hide mandatory controls. Every flag must have an owner, default state, introduction version, removal criterion, and tests.

## 8. Configuration tests

| ID | Test | Expected result |
|---|---|---|
| CFG-01 | First start without config | Secure defaults |
| CFG-02 | Valid config | Successful load |
| CFG-03 | Malformed config | Clear error, safe fallback |
| CFG-04 | Unknown field | Documented compatibility policy |
| CFG-05 | Out-of-range value | Rejection |
| CFG-06 | Override of a mandatory constraint | Rejection |
| CFG-07 | Valid config reload | Atomic application |
| CFG-08 | Failed reload | Last valid config preserved |
| CFG-09 | Secret in config file | Not allowed, or migration to Credential Store |
| CFG-10 | Telegram default | Disabled |
| CFG-11 | Auto-response default | Disabled |
| CFG-12 | Policy change audit | Audit event present |
| CFG-13 | Config backup | Config restorable |
| CFG-14 | Config rollback | Previous version restored |
| CFG-15 | Restart required | UI reports the need and status |

## 9. Open questions

- Runtime source of truth for each setting.
- Config format (TOML/YAML/JSON) and schema.
- Local policy and possible profiles.
- Multi-version config handling.
- Secret store for each future platform.

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

### Configuration of multi-language components

The configuration of Rust/Python workers must be subordinate to the policy and configuration validated by the Go service. Workers cannot redefine security controls, enable remote listeners, or autonomously modify response policies. Packaging/runtime-specific parameters are separate from user preferences and must not contain secrets.
