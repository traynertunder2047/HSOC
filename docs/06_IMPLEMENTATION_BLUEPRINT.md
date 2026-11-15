# HSOC — Implementation Blueprint

**Versione:** 1.1-draft | **Obiettivo:** tradurre le specifiche in moduli implementabili

## 1. Stack di riferimento

- Go: servizio, IPC, policy, raccolta, normalizzazione, detection, storage, backup e audit.
- Tauri + React + TypeScript: desktop UI.
- SQLite: storage locale, accessibile in scrittura solo da Go.
- Rust: moduli specialistici di sicurezza, con confine FFI/IPC esplicito.
- Python: worker e attività ausiliarie di analisi, con input/output schema-validati.
- CI: workflow distinti per Go, Rust, Python e dashboard, sui componenti presenti.

Go + Rust + Python sono lo stack approvato. L'integrazione è graduale e guidata da requisiti; ogni runtime ha responsabilità distinte e un piano di packaging/manutenzione.

## 2. Struttura repository proposta

```text
hsoc/
├── README.md
├── LICENSE
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
├── go.sum
├── cmd/
│   └── hsoc-service/
│       └── main.go
├── internal/
│   ├── service/
│   ├── ipc/
│   ├── config/
│   ├── collector/
│   ├── normalizer/
│   ├── inventory/
│   ├── detection/
│   ├── incidents/
│   ├── response/
│   ├── storage/
│   ├── backup/
│   ├── notifications/
│   ├── audit/
│   └── diagnostics/
├── rust/           # moduli specialistici di sicurezza
├── python/         # worker e analisi ausiliarie
├── database/
│   ├── schema/
│   ├── migrations/
│   └── queries/
├── protocols/      # IPC, eventi, messaggi e schemi
├── desktop/
│   ├── src/        # React + TypeScript
│   └── src-tauri/  # shell Tauri, distinta dai moduli rust/
├── tests/
│   ├── fixtures/
│   ├── integration/
│   ├── system/
│   ├── security/
│   └── recovery/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   └── operations/
├── scripts/
└── .github/workflows/
```

## 3. Dipendenze tra moduli

- `cmd` avvia `service`.
- `service` coordina moduli interni.
- `ipc` espone solo use case autorizzati.
- `storage` è l'unico livello che parla con SQLite.
- `collector` non decide la severità; produce dati grezzi/normalizzati.
- `normalizer` valida e mappa eventi canonici.
- `detection` valuta regole su eventi normalizzati.
- `incidents` aggrega rilevamenti.
- `response` applica policy e approvazioni; in MVP solo simulazione.
- `audit` registra modifiche sensibili.
- `ui` parla solo via IPC.

Evitare dipendenze cicliche e accesso globale al DB.

## 4. Vertical slice obbligatoria

Prima di implementare molte feature, realizzare una sola catena completa e sicura:

1. acquisire un evento di test da fixture o fonte OS non distruttiva;
2. validarlo e normalizzarlo in Go;
3. salvarlo in SQLite tramite repository Go;
4. valutarlo con una regola demo deterministica;
5. creare un detection record e un incident;
6. esporre i dati via IPC;
7. visualizzarli in dashboard;
8. registrare log e audit dove necessario;
9. testare riavvio e recovery.

Nessuna risposta automatica in questa slice.

## 5. Regole di implementazione

- Interfacce piccole e testabili.
- Context/cancellation e timeout per I/O.
- Errori tipizzati e messaggi redatti.
- Config validata all'avvio e su ogni update.
- No shell execution derivata da input.
- No goroutine senza lifecycle/stop path.
- DB access solo tramite repository.
- Migrazioni separate dal codice applicativo.
- Fixture sintetiche e riproducibili.
- Feature flag per funzioni non pronte.
- Dipendenze bloccate a versioni e aggiornate tramite PR.

## 6. Strategia linguaggi

**Go**: core V1, orchestrazione, policy, persistenza e unico writer SQLite.  
**Rust**: moduli specialistici di sicurezza, con interfacce ristrette, test, audit dipendenze e packaging verificato.  
**Python**: worker e analisi ausiliarie; input/output schema-validati, ambiente e dipendenze bloccati, isolamento e gestione del fallimento. Non è autorità sullo stato o sul database.

## 7. CI minima

1. Go: `gofmt` check, `go vet`, `go test ./...`, `govulncheck`.
2. Rust: `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test`, `cargo audit`.
3. Python: `ruff check`, `ruff format --check`, `pytest`, `pip-audit`.
4. Dashboard: lint, TypeScript typecheck, test e build.
5. Integration test con DB temporaneo e migrazioni.
6. Dependency vulnerability scan e secret scan.
7. Build riproducibile e report associati al commit.

## 8. Sequenza di implementazione

1. Repo, logging, config e build.
2. Schema DB e migrations.
3. Service lifecycle e health.
4. IPC e autorizzazione.
5. Primo collector e normalizer.
6. Detection rule engine.
7. Incidents e audit.
8. Dashboard.
9. Backup/restore.
10. Inventory e vulnerability checks.
11. Notifications opt-in.
12. Hardening, installer e release.

## 9. Test implementazione

| ID | Area | Esito atteso |
|---|---|---|
| IMP-01 | Build core | Build riproducibile |
| IMP-02 | Moduli isolati | Unit test passano |
| IMP-03 | Dipendenze | Nessun import ciclico architetturale |
| IMP-04 | DB access | Solo storage package apre DB |
| IMP-05 | UI | Nessun accesso diretto a file DB |
| IMP-06 | Config invalid | Fail safe |
| IMP-07 | Lifecycle | Stop pulito senza goroutine residue |
| IMP-08 | CI | Pipeline fallisce su test/security gate |
| IMP-09 | Fixture | Stesso input produce stesso risultato |
| IMP-10 | Worker assente | Core degrada in modo controllato |

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

### Struttura monorepo di riferimento

```text
HSOC/
├── .github/workflows/       # CI Go/Rust/Python/dashboard, security, release
├── cmd/                     # entry point Go
├── internal/                # core Go e moduli applicativi
├── rust/                    # moduli specialistici di sicurezza
├── python/                  # worker e analisi ausiliarie
├── desktop/                 # Tauri + React + TypeScript
│   └── src-tauri/           # shell Tauri; distinto dai moduli rust/
├── database/                # schema, migrations, queries
├── protocols/               # IPC, eventi, messaggi e schemi
├── rules/                   # detection, correlazione, IoT, vulnerabilità
├── configs/                 # default e configurazioni piattaforma
├── installers/windows/      # packaging iniziale
├── scripts/                 # build, test, sviluppo e release
├── tests/                   # unit, integration, system, security, recovery
├── docker/                  # solo ambienti facoltativi sviluppo/test
├── docs/                    # documentazione e ADR
├── README.md
├── LICENSE                  # MIT
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
└── go.sum
```

Le cartelle dei componenti vanno create quando si avvia il relativo modulo, non come directory vuote prive di responsabilità. Gli agenti remoti V2 avranno una sezione futura dedicata e non devono essere implementati nella V1.
