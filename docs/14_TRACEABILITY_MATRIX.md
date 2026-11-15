# HSOC — Matrice completa di tracciabilità

**Versione:** 1.1-draft | **Data:** 2026-11-15  
**Nota:** requisiti pianificati; gli stati iniziali sono `DA IMPLEMENTARE` e `DA TESTARE`, salvo evidenza verificabile.

## 1. Convenzioni

- ID stabili: non riutilizzare un ID ritirato.
- Priorità: P0 essenziale MVP; P1 importante V1; P2 facoltativo; P3 V2+.
- Versioni: V1-MVP, V1, V2+.
- Stato: `SPECIFICATO`, `DA CONFERMARE`, `DA IMPLEMENTARE`, `DA TESTARE`, `SUPERATO`.
- Ogni requisito deve collegarsi a documento, componente e almeno un test.
- I riferimenti ai test sono identificatori da materializzare in test case eseguibili; non attestano esecuzione.

## 2. Matrice requisiti


### 2.1 Requisiti FUN

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-FUN-001 | Agente locale Go | 01,06 | `service` | SYS-001,IMP-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-002 | Servizio in background | 01,05,06 | `service` | SYS-001,DEP-06 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-003 | Raccolta eventi Windows | 01,06,10 | `collector` | INT-02,DET-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-004 | Validazione e normalizzazione | 01,06,10 | `normalizer` | INT-02,DET-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-005 | Persistenza eventi SQLite | 02,06 | `storage` | DB-001,DB-006 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-006 | Motore regole detection | 01,10 | `detection` | DET-01..15 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-007 | Deduplicazione | 02,10 | `detection/storage` | DB-005,DET-04 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-008 | Correlazione eventi | 10 | `detection/incidents` | DET-05,DET-06 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-009 | Gestione incidenti | 01,10 | `incidents` | INT-04,UI-02 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-010 | Dashboard desktop | 01,03,08 | `ui` | INT-05,UI-01..10 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-011 | Notifiche locali | 01,08 | `notifications` | INT-08,UI-05 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-012 | Telegram opt-in | 01,08,11 | `notifications` | CFG-10,SEC-05 | P2 | V1 | DA IMPLEMENTARE |
| TR-FUN-013 | Inventario software | 01,09 | `inventory` | INV-01..05 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-014 | Controllo vulnerabilità mensile | 01,09 | `vulnerability` | MNT-01,INV-06 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-015 | Backup e restore | 01,02,05,08 | `backup/storage` | DB-009..14,REC-01..08 | P0 | V1 | DA IMPLEMENTARE |
| TR-FUN-016 | Configurazione e policy | 01,11 | `config` | CFG-01..15 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-FUN-017 | Health monitor | 01,08,12 | `diagnostics` | SYS-001,UI-01 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-018 | Diagnostica esportabile | 04,08,12 | `diagnostics` | SEC-09,UI-07 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-019 | Modalità simulazione | 10,12 | `response` | DET-11 | P1 | V1 | DA IMPLEMENTARE |
| TR-FUN-020 | Audit operazioni sensibili | 02,04,10,11 | `audit` | SEC-12,CFG-12 | P0 | V1-MVP | DA IMPLEMENTARE |

### 2.2 Requisiti ARC

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-ARC-001 | Go autorità centrale | 01,06,12 | `service` | IMP-01,IMP-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-002 | Solo Go scrive SQLite | 02,03,06,13 | `storage` | IMP-04,SEC-11 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-003 | Separazione dashboard/servizio | 01,03,06 | `ipc/ui` | INT-05,SEC-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-004 | IPC locale autorizzato | 03,04 | `ipc` | IPC-001..12 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-005 | Contratti IPC versionati | 03 | `ipc/schemas` | IPC-006 | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-006 | Worker Python isolati per analisi ausiliarie, senza accesso DB | 01,02,06,12 | `python/` | IMP-09, QA worker | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-007 | Moduli Rust specialistici di sicurezza, senza accesso DB | 01,04,06,12 | `rust/` | IMP-10, SEC tests | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-013 | Dashboard Tauri + React + TypeScript | 01,03,06,08 | `desktop/` | UI-01..10 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-014 | Monorepo GitHub con CI per ogni stack | 06,07,13,16 | `.github/workflows/` | CI report | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-015 | Licenza progetto MIT e avvisi terze parti | 06,09,13,16 | `LICENSE`, `THIRD_PARTY_NOTICES.md` | license review | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-016 | Docker solo facoltativo per sviluppo/test | 05,06,16 | `docker/` | build senza Docker | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-008 | Gestione lifecycle moduli | 06 | `service` | IMP-07,IMP-09 | P1 | V1 | DA IMPLEMENTARE |
| TR-ARC-009 | Agenti remoti esclusi V1 | 01,12,13 | `scope` | review architetturale | P0 | V1 | DA IMPLEMENTARE |
| TR-ARC-010 | Protocollo futuro versionabile | 03 | `schemas/ipc` | IPC-006 | P2 | V2+ | DA IMPLEMENTARE |
| TR-ARC-011 | Dipendenze a direzione controllata | 06,13 | `internal packages` | IMP-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-ARC-012 | Nessun listener remoto default | 04,11 | `service/config` | SEC-01,CFG-01 | P0 | V1-MVP | DA IMPLEMENTARE |

### 2.3 Requisiti DB

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-DB-001 | SQLite locale | 02 | `storage` | DB-001 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-002 | Foreign key abilitate | 02 | `storage` | DB-004 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-003 | Transazioni multi-entità | 02 | `storage` | DB-006 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-004 | Migrazioni versionate | 02,09 | `migrations` | DB-002,DB-003 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-005 | Integrità dopo crash | 02,07 | `storage` | DB-013,REC-05 | P0 | V1 | DA IMPLEMENTARE |
| TR-DB-006 | Dedup fingerprint | 02,10 | `events/detections` | DB-005,DET-04 | P1 | V1 | DA IMPLEMENTARE |
| TR-DB-007 | Retention configurabile | 02,11 | `storage/config` | DB-011,DB-012 | P1 | V1 | DA IMPLEMENTARE |
| TR-DB-008 | Segreti fuori DB | 02,04,11 | `credential store` | SEC-05,CFG-09 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-009 | Audit persistente | 02,04 | `audit_log` | SEC-12 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-010 | Backup compatibile WAL | 02,05 | `backup` | DB-009,REC-01 | P0 | V1 | DA IMPLEMENTARE |
| TR-DB-011 | Timestamp UTC | 02,03 | `storage` | DB-001,INT-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DB-012 | Eliminazione retention auditata | 02,11 | `storage/audit` | DB-011,DB-012 | P1 | V1 | DA IMPLEMENTARE |

### 2.4 Requisiti SEC

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-SEC-001 | Minimo privilegio | 01,04,05,13 | `service/installer` | SEC-06,SEC-14 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-002 | Nessun login HSOC dedicato | 01,03 | `ipc` | IPC-001 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-003 | IPC protetto | 03,04 | `ipc` | SEC-01,IPC-002 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-004 | Validazione input | 03,04,13 | `all boundaries` | SEC-02..04 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-005 | Gestione sicura credenziali | 02,04,08,11 | `credential store` | SEC-05,CFG-09 | P0 | V1 | DA IMPLEMENTARE |
| TR-SEC-006 | Update verificati | 04,05,09 | `updater` | SEC-08,DEP-03 | P1 | V1 | DA IMPLEMENTARE |
| TR-SEC-007 | Autorizzazione response | 04,10 | `response` | DET-12,DET-13 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-008 | Audit operazioni sensibili | 02,04,10 | `audit` | SEC-12 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-009 | Minimizzazione dati | 01,04,08 | `collectors/diagnostics` | SEC-09,UI-07 | P0 | V1 | DA IMPLEMENTARE |
| TR-SEC-010 | Notifiche senza dati eccessivi | 04,08,11 | `notifications` | SEC-05,CFG-10 | P1 | V1 | DA IMPLEMENTARE |
| TR-SEC-011 | Isolamento moduli opzionali | 04,06 | `worker boundary` | SEC-10,IMP-09 | P1 | V1 | DA IMPLEMENTARE |
| TR-SEC-012 | Gestione errori DB sicura | 02,04 | `storage` | DB-006..08,SEC-11 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-013 | No comandi arbitrari | 03,04,13 | `ipc/response` | SEC-04,IPC-010 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-SEC-014 | Protezione backup | 02,04,05 | `backup` | DB-009,REC-02 | P0 | V1 | DA IMPLEMENTARE |
| TR-SEC-015 | Privacy local-first | 01,08,11 | `all components` | UI-07,CFG-10 | P0 | V1 | DA IMPLEMENTARE |
| TR-SEC-016 | Revoca agenti remoti futura | 01,04 | `future coordinator` | FUT-004 | P3 | V2+ | DA IMPLEMENTARE |

### 2.5 Requisiti DET

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-DET-001 | Separare event/detection/incident | 02,10 | `events/detection/incidents` | INT-04,DET-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-002 | Severità e confidence distinte | 08,10 | `detection/ui` | DET-11,DET-12,UI-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-003 | Motivazione ed evidenza | 08,10 | `detection` | DET-01,UI-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-004 | Test FP/FN per regola | 07,10 | `tests/fixtures` | DET-02,DET-15 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-005 | Versionamento regole | 02,10 | `rules` | DET-08 | P1 | V1 | DA IMPLEMENTARE |
| TR-DET-006 | Dedup/correlazione | 02,10 | `detection` | DET-04..06 | P1 | V1 | DA IMPLEMENTARE |
| TR-DET-007 | Workflow incidenti | 08,10 | `incidents` | INT-04,UI-02 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-008 | Risposta ibrida conservativa | 01,10 | `response` | DET-11..13 | P0 | V1 | DA IMPLEMENTARE |
| TR-DET-009 | Approvazione azioni sensibili | 04,10 | `response/approvals` | DET-12,DET-13 | P0 | V1 | DA IMPLEMENTARE |
| TR-DET-010 | Auto-response OFF default | 04,10,11 | `response/config` | DET-12,CFG-11 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-011 | Simulation senza effetti | 10,12 | `response` | DET-11 | P1 | V1 | DA IMPLEMENTARE |
| TR-DET-012 | Audit response | 02,04,10 | `audit` | SEC-12,DET-13 | P0 | V1 | DA IMPLEMENTARE |
| TR-DET-013 | Fail-safe in incertezza | 04,10 | `response` | DET-09,SEC-11 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DET-014 | Metriche detection | 07,10 | `qa/metrics` | DET-15 | P1 | V1 | DA IMPLEMENTARE |
| TR-DET-015 | IoT passivo e limitato | 01,08,10 | `iot observer` | UI-08 | P1 | V1 | DA IMPLEMENTARE |
| TR-DET-016 | Feedback falso positivo | 08,10 | `incidents/detection` | DET-10 | P1 | V1 | DA IMPLEMENTARE |

### 2.6 Requisiti CFG

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-CFG-001 | Default sicuri | 01,11 | `config` | CFG-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-CFG-002 | Config schema-validata | 03,11 | `config` | CFG-02..05 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-CFG-003 | Precedence documentata | 11,12 | `config` | CFG-06 | P1 | V1 | DA IMPLEMENTARE |
| TR-CFG-004 | Vincoli sicurezza non disattivabili | 04,11 | `policy` | CFG-06,SEC-07 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-CFG-005 | Moduli opzionali disattivati | 06,11 | `config` | IMP-09,CFG-01 | P1 | V1 | DA IMPLEMENTARE |
| TR-CFG-006 | Separazione config/DB/segreti | 02,11 | `config/storage` | CFG-09,DB-008 | P0 | V1 | DA IMPLEMENTARE |
| TR-CFG-007 | Policy response | 10,11 | `response/config` | CFG-11,DET-12 | P0 | V1 | DA IMPLEMENTARE |
| TR-CFG-008 | Retention | 02,11 | `storage/config` | DB-011 | P1 | V1 | DA IMPLEMENTARE |
| TR-CFG-009 | Notifiche opt-in | 08,11 | `notifications` | CFG-10 | P1 | V1 | DA IMPLEMENTARE |
| TR-CFG-010 | Preservare config valida | 11 | `config` | CFG-03,CFG-08 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-CFG-011 | Audit config | 02,11 | `audit` | CFG-12 | P0 | V1 | DA IMPLEMENTARE |
| TR-CFG-012 | Credential Store OS | 04,08,11 | `credential store` | SEC-05,CFG-09 | P0 | V1 | DA IMPLEMENTARE |
| TR-CFG-013 | Backup configurazione | 02,11 | `backup` | CFG-13 | P1 | V1 | DA IMPLEMENTARE |
| TR-CFG-014 | Nessun remote agent V1 | 01,11,12 | `scope` | review | P0 | V1 | DA IMPLEMENTARE |
| TR-CFG-015 | Rollback config | 11 | `config` | CFG-14 | P1 | V1 | DA IMPLEMENTARE |

### 2.7 Requisiti DEP

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-DEP-001 | Windows x86-64 iniziale | 01,05,12 | `installer` | DEP-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-002 | Versioni Windows dichiarate | 05,12 | `compatibility matrix` | DEP-02 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-003 | Installazione servizio/UI | 05 | `installer` | DEP-01,DEP-05 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-004 | Directory e ACL | 04,05 | `installer` | DEP-04,SEC-06 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-005 | Inizializzazione DB | 02,05 | `bootstrap` | DEP-05,DB-001 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-006 | Health check installazione | 05,12 | `service` | DEP-05,SYS-001 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-007 | Update firmati | 04,05,09 | `updater` | DEP-03,SEC-08 | P1 | V1 | DA IMPLEMENTARE |
| TR-DEP-008 | Rollback update | 05,09 | `updater/backup` | DEP-09,REC-06 | P1 | V1 | DA IMPLEMENTARE |
| TR-DEP-009 | Disinstallazione controllata | 05 | `installer` | DEP-11,DEP-12 | P1 | V1 | DA IMPLEMENTARE |
| TR-DEP-010 | Nessun hardware esterno | 01,05 | `product scope` | DEP-14 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEP-011 | Linux futuro | 01,05 | `platform adapters` | FUT-007 | P3 | V2+ | DA IMPLEMENTARE |
| TR-DEP-012 | macOS futuro | 01,05 | `platform adapters` | FUT-008 | P3 | V2+ | DA IMPLEMENTARE |
| TR-DEP-013 | Reinstallazione con dati esistenti | 05 | `installer/storage` | DEP-13 | P1 | V1 | DA IMPLEMENTARE |
| TR-DEP-014 | Agenti remoti futuri | 01,05,12 | `coordinator` | FUT-001..06 | P3 | V2+ | DA IMPLEMENTARE |

### 2.8 Requisiti QA

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-QA-001 | Unit test | 07,13 | `tests/unit` | CI unit | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-002 | Integration test | 07 | `tests/integration` | INT-01..10 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-003 | System test | 07 | `tests/system` | SYS-001.. | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-004 | Security test | 04,07 | `tests/security` | SEC-01..16 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-005 | Regression test | 07,10 | `tests/regression` | DET-15 | P0 | V1 | DA IMPLEMENTARE |
| TR-QA-006 | Performance test | 07 | `tests/performance` | benchmark report | P1 | V1 | DA IMPLEMENTARE |
| TR-QA-007 | Recovery test | 02,05,07 | `tests/recovery` | REC-01..08 | P0 | V1 | DA IMPLEMENTARE |
| TR-QA-008 | Detection quality test | 07,10 | `tests/detection` | DET-15 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-009 | CI pipeline | 06,07,13 | `workflows` | CI report | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-010 | Definition of Ready | 12,13 | `process` | review checklist | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-011 | Definition of Done | 12,13 | `process` | PR checklist | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-012 | Bug severity classification | 07,09 | `triage` | bug records | P1 | V1 | DA IMPLEMENTARE |
| TR-QA-013 | Release gates | 07,09,12 | `release` | GO/NO-GO record | P0 | V1 | DA IMPLEMENTARE |
| TR-QA-014 | Evidence retention | 07 | `test reports` | test artifacts | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-QA-015 | Test environment isolation | 07,12 | `test infra` | environment report | P1 | V1 | DA IMPLEMENTARE |

### 2.9 Requisiti MNT

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-MNT-001 | Scan dipendenze mensile | 09,13 | `CI/maintenance` | MNT-01 | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-002 | Advisory critici fuori ciclo | 09 | `security process` | MNT-02 | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-003 | SBOM per release | 09 | `release pipeline` | MNT-03 | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-004 | Semantic versioning | 09 | `release` | release review | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-005 | Canali release | 09 | `distribution` | release checklist | P2 | V1 | DA IMPLEMENTARE |
| TR-MNT-006 | Migrazioni immutabili/versionate | 02,09 | `migrations` | DB-002, MNT-07 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-MNT-007 | ADR mantenute | 01,12 | `docs/decisions` | ADR review | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-008 | Patch con review | 09,13 | `PR process` | MNT-10 | P0 | V1 | DA IMPLEMENTARE |
| TR-MNT-009 | Known issues e changelog | 08,09 | `release docs` | MNT-08 | P1 | V1 | DA IMPLEMENTARE |
| TR-MNT-010 | Auto-fix solo proposta revisionata | 09,13 | `maintenance` | MNT-10 | P0 | V1 | DA IMPLEMENTARE |

### 2.10 Requisiti UI

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-UI-001 | Dashboard stato | 01,08 | `ui` | UI-01 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-UI-002 | Distinguere eventi/detection/incidenti | 08,10 | `ui` | UI-02 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-UI-003 | Mostrare severity/confidence | 08,10 | `ui` | UI-03 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-UI-004 | Health e diagnostica | 08,12 | `ui` | UI-01,UI-07 | P1 | V1 | DA IMPLEMENTARE |
| TR-UI-005 | Configurazione sicura | 08,11 | `ui` | UI-05 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-UI-006 | Gestione errori UI | 03,08 | `ui/ipc` | UI-04 | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-UI-007 | Guide utente/tecniche | 08,09 | `docs` | UI-09,UI-10 | P1 | V1 | DA IMPLEMENTARE |
| TR-UI-008 | Limiti IoT dichiarati | 01,08,10 | `ui/docs` | UI-08 | P0 | V1 | DA IMPLEMENTARE |
| TR-UI-009 | Known issues visibili | 08,09 | `ui/docs` | MNT-08 | P1 | V1 | DA IMPLEMENTARE |
| TR-UI-010 | Conferma restore e impatto | 02,05,08 | `ui/backup` | UI-06 | P0 | V1 | DA IMPLEMENTARE |

### 2.11 Requisiti FUT

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-FUT-001 | Agenti remoti | 01,12 | `future/agents` | architecture review | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-002 | Identità agente | 04 | `future/identity` | security design | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-003 | Enrollment autenticato | 04 | `future/enrollment` | security design | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-004 | Revoca agente | 04 | `future/identity` | security test | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-005 | Sincronizzazione | 01,03 | `future/sync` | integration test | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-006 | Task remoti sicuri | 04,10 | `future/tasks` | threat model | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-007 | Linux | 01,05 | `platform/linux` | platform suite | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-008 | macOS | 01,05 | `platform/macos` | platform suite | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-009 | Coordinatore distribuito | 01 | `future/coordinator` | system test | P3 | V2+ | DA IMPLEMENTARE |
| TR-FUT-010 | Policy distribuite | 11 | `future/policy` | security/integration | P3 | V2+ | DA IMPLEMENTARE |

### 2.12 Requisiti repository e DevSecOps

| ID | Requisito | Doc | Componente | Test/evidenza | Priorità | Versione | Stato iniziale |
|---|---|---|---|---|---|---|---|
| TR-DEV-001 | GitHub Actions per Go, Rust, Python e dashboard | 06,07,13,16 | `.github/workflows/` | CI report | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEV-002 | Branch main protetto e PR obbligatorie | 13,16 | repository settings | branch protection evidence | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEV-003 | Secret scanning e permessi minimi workflow | 04,13,16 | GitHub settings/workflows | security config review | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEV-004 | Docker non richiesto per installazione utente | 05,06,16 | installer/build | install test senza Docker | P0 | V1-MVP | DA IMPLEMENTARE |
| TR-DEV-005 | MIT e inventario licenze dipendenze | 09,13,16 | LICENSE/SBOM/notices | license review | P1 | V1 | DA IMPLEMENTARE |

## 3. Test trasversali da mantenere

| ID test | Area | Requisiti principali | Evidenza richiesta |
|---|---|---|---|
| SYS-001..003 | End-to-end | TR-FUN-001..010 | log, DB fixture, screenshot UI |
| INT-01..10 | Integrazione | TR-ARC, TR-FUN, TR-DB | report automatico |
| SEC-01..16 | Sicurezza | TR-SEC-001..016 | report security test |
| DET-01..15 | Detection | TR-DET-001..016 | dataset, metriche, report |
| CFG-01..15 | Configurazione | TR-CFG-001..015 | fixture e risultato |
| DB-001..14 | Database | TR-DB-001..012 | DB temporaneo e integrity check |
| DEP-01..14 | Deployment | TR-DEP-001..014 | VM/OS test matrix |
| REC-01..08 | Recovery | TR-FUN-015, TR-DB-005/010 | backup, restore e checksum |
| UI-01..10 | Usabilità | TR-UI-001..010 | test UI e review |
| MNT-01..10 | Manutenzione | TR-MNT-001..010 | CI, SBOM, advisory report |
| IMP-01..10 | Implementazione | TR-ARC, TR-QA | CI e architettura |

## 4. Modello scheda test

Ogni test case deve contenere:
- `test_id`
- `requirement_ids`
- nome e obiettivo
- ambiente, OS, versione HSOC e commit
- prerequisiti
- fixture/input
- passi numerati
- risultato atteso
- risultato effettivo
- stato (`NOT RUN`, `PASS`, `FAIL`, `BLOCKED`, `SKIPPED`)
- data, autore e link/evidenza
- bug associati

## 5. Questioni da chiudere prima del freeze

1. Specifica fisica SQL completa e indici.
2. `policy_assignments` e relazioni `actions`/`approvals`.
3. Semantica UNIQUE/dedup con valori NULL.
4. IPC Named Pipes: ACL, identità client e protocollo.
5. Versioni Windows supportate.
6. Fonte di verità config/runtime e formato config.
7. Retention e limiti di spazio.
8. Credential Store e protezione backup.
9. Budget performance su hardware di riferimento.
10. Matrice fonti eventi effettivamente disponibili.

## 6. Regole di aggiornamento

- Quando un requisito cambia, aggiornare documento origine e questa matrice nella stessa PR.
- Ogni requisito P0 deve avere test P0.
- Se un test fallisce, aprire bug collegato a requisito e test.
- Un requisito non testabile va riscritto o giustificato.
- Prima della release, generare snapshot della matrice con commit/versione.
- Non trasformare automaticamente `SPECIFICATO` in `IMPLEMENTATO` o `PASS`.
- La copertura si calcola separatamente per specifica, implementazione e test superati.

## 7. Indicatori di avanzamento

- Copertura specifica = requisiti con documento valido / requisiti totali.
- Copertura implementazione = requisiti con implementazione verificata / requisiti totali.
- Copertura test = requisiti con almeno un test eseguito / requisiti totali.
- Pass rate = test PASS / test eseguiti (escludendo NOT RUN/BLOCKED/SKIPPED).
- Rischio residuo = requisiti P0 senza test PASS + bug aperti per severità.

Questi indicatori non vanno aggregati in un singolo numero che nasconda lacune critiche.

## Aggiornamento architetturale 1.1 – decisioni consolidate (15-11-2026)

Le seguenti decisioni prevalgono su eventuali formulazioni precedenti in questo documento:

- **Licenza del progetto:** MIT. Il testo completo è nel file `LICENSE` alla radice del repository. Le licenze delle dipendenze restano da verificare e rispettare separatamente.
- **Repository:** monorepo GitHub, con codice, documentazione, test e configurazioni di build nello stesso repository.
- **Dashboard desktop:** Tauri + React + TypeScript. Il runtime Rust interno a Tauri è distinto dai moduli Rust specialistici HSOC.
- **Stack applicativo:** Go + Rust + Python. Go è il core e l'unica autorità di scrittura su SQLite; Rust implementa componenti specialistici di sicurezza; Python implementa worker e attività ausiliarie di analisi. Le responsabilità devono restare separate e i confini tra processi/moduli documentati.
- **Introduzione graduale:** i tre linguaggi fanno parte dello stack approvato, ma le rispettive componenti vengono integrate per fasi verticali verificabili; non si creano cartelle o moduli vuoti e un worker non deve diventare un punto singolo di guasto del core.
- **Docker:** facoltativo, riservato ad ambienti di sviluppo/test o servizi ausiliari isolati. Non è prerequisito di installazione o funzionamento per l'utente HSOC.
- **Agenti remoti:** esclusi dalla V1; progettazione e implementazione funzionale dalla V2+. Nella V1 si possono definire contratti e schemi versionati per evitare una futura riprogettazione, senza aprire listener remoti.
- **Piattaforma iniziale:** Windows x86-64; Linux e macOS restano evoluzioni successive.
- **Tracciabilità:** requisiti, documenti, componenti, test ed evidenze devono essere collegati nella matrice del Documento 14.

Queste decisioni aggiornano la baseline progettuale, non attestano che codice, workflow CI, installer o test siano già stati implementati o verificati.

### Aggiornamento tracciabilità 1.1

I requisiti `TR-ARC-006` e `TR-ARC-007` non classificano più Python e Rust come linguaggi opzionali: sono componenti approvati dello stack con ruoli distinti. La loro implementazione resta tracciata come `DA IMPLEMENTARE` finché non esistono codice e prove. Sono stati aggiunti requisiti per dashboard, monorepo/CI, MIT, Docker facoltativo e controlli DevSecOps.
