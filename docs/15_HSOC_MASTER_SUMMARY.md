# HSOC — Riepilogo generale completo

**Versione:** 1.1-draft | **Data:** 2026-11-15  
**Scopo:** documento di orientamento unico per sviluppo, test e manutenzione.

## 1. Visione

HSOC è un centro di sicurezza informatica domestico, desktop e local-first. La V1 protegge e osserva principalmente il computer su cui è installato; la visibilità IoT è limitata ai dati accessibili senza hardware aggiuntivo e senza scansioni intrusive.

Obiettivo: rendere disponibili inventario, eventi, rilevamenti, incidenti, stato e backup in un'interfaccia comprensibile, senza costi obbligatori di server/cloud.

## 2. Vincoli confermati

- Costo obiettivo: €0 per il funzionamento base.
- Nessun hardware esterno richiesto.
- Nessuna telecamera o sensore obbligatorio.
- Nessun login HSOC dedicato nella V1.
- Desktop, modalità mista: servizio background + dashboard on demand.
- Windows prima; Linux/macOS dopo.
- Un solo agente principale in V1; agenti remoti in V2+.
- Risposta ibrida conservativa.
- Privilegi minimi.
- Dati locali con backup.
- Telegram opzionale.
- Controllo vulnerabilità mensile e fuori ciclo per criticità.
- Auto-fix soltanto come patch proposta, testata e revisionata da una persona.

## 3. Funzionalità

### Core
- Lifecycle servizio e health monitor.
- Raccolta eventi OS supportati.
- Validazione e normalizzazione.
- Storage SQLite.
- Regole detection versionate.
- Dedup e correlazione.
- Incident management.
- Dashboard.
- Audit e diagnostica.

### Sicurezza e manutenzione
- Inventario software.
- Vulnerability checks.
- Dependency scan e SBOM.
- Backup/restore.
- Firma e verifica aggiornamenti.
- Config e policy validate.
- Test e metriche detection.

### Opzionali
- Telegram, disattivato inizialmente.
- Moduli Rust specialistici di sicurezza e worker Python di analisi ausiliaria, integrati per fasi.
- Simulazione di risposta.

### Future V2+
- Agenti remoti.
- Enrollment, identità e revoca.
- Coordinatore e sync.
- Linux e macOS.
- Policy distribuite.

## 4. Architettura

```text
Tauri + React + TypeScript
        |
        | IPC locale autorizzato
        v
HSOC Service (Go)
  |-- service/lifecycle/health
  |-- collector + normalizer
  |-- inventory + vulnerability
  |-- detection + incidents
  |-- response + approvals
  |-- config + policy
  |-- audit + diagnostics
  |-- notifications
  |-- backup/restore
  |
  v
SQLite (scrittura esclusiva Go)

OS adapters: Windows first
Credential Store OS: segreti
Rust: moduli specialistici di sicurezza; Python: worker/analisi ausiliarie; entrambi isolati e senza accesso diretto al DB
```

## 5. Stack linguaggi

| Linguaggio/tecnologia | Ruolo | Obbligatorietà |
|---|---|---|
| Go | Core e servizio | Obbligatorio |
| Tauri | Shell desktop | Obbligatorio per UI desktop |
| React | UI | Obbligatorio nella proposta |
| TypeScript | UI tipizzata | Obbligatorio nella proposta |
| SQLite | Database | Obbligatorio V1 |
| Rust | Moduli specialistici di sicurezza | Linguaggio approvato; integrazione per fasi |
| Python | Worker e analisi ausiliarie | Linguaggio approvato; integrazione per fasi |
| SQL | Schema e query | Necessario per storage |

Regola: non aggiungere runtime solo per preferenza. Ogni linguaggio extra deve risolvere un requisito e avere un costo di packaging/manutenzione giustificato.

## 6. Struttura repository

```text
HSOC/
├── .github/workflows/       # CI Go/Rust/Python/dashboard, security, release
├── cmd/                     # entry point Go
├── internal/                # core Go e moduli applicativi
├── rust/                    # moduli specialistici di sicurezza
├── python/                  # worker e analisi ausiliarie
├── desktop/
│   ├── src/                 # React + TypeScript
│   └── src-tauri/           # shell Tauri
├── database/                # schema, migrations, queries
├── protocols/               # IPC, eventi, messaggi e schemi
├── rules/                   # detection, correlazione, IoT, vulnerabilità
├── configs/                 # default e configurazioni piattaforma
├── installers/windows/      # packaging iniziale
├── scripts/                 # build, test, sviluppo e release
├── tests/                   # unit, integration, system, security, recovery
├── docker/                  # facoltativo: sviluppo/test
├── docs/                    # documentazione e ADR
├── README.md
├── LICENSE                  # MIT
├── SECURITY.md
├── CONTRIBUTING.md
├── go.mod
└── go.sum
```

La V1 è locale e Windows-first. Le cartelle si creano quando i relativi moduli vengono implementati; non si aggiungono directory vuote. Gli agenti remoti sono una funzionalità V2+.

## 7. Regole di sviluppo essenziali

1. Go è l'autorità centrale.
2. Solo Go scrive SQLite.
3. UI comunica via IPC e non legge file DB.
4. IPC protetto da ACL e autorizzazione per operazione.
5. Minimo privilegio.
6. Input sempre validato.
7. Nessun comando arbitrario da input o notifiche.
8. Segreti solo in Credential Store.
9. Telegram e auto-response OFF per default.
10. Agenti remoti esclusi dalla V1.
11. Ogni regola detection ha test positivi e negativi.
12. Ogni modifica aggiorna test, documentazione e matrice.
13. CI verde e review prima del merge.
14. Nessuna dichiarazione PASS senza evidenza.

Consultare `13_DEVELOPMENT_RULES.md` per il regolamento completo.

## 8. Strategia test

- Unit: funzioni, parser, regole, config, repository.
- Integration: IPC, DB, migrazioni, notifiche mock.
- System: flussi end-to-end e UI.
- Security: ACL, input, segreti, update, least privilege.
- Recovery: backup, restore, crash, disk full.
- Performance: CPU/RAM, throughput, latenza e crescita DB.
- Detection quality: TP/FP/TN/FN, precision, recall, F1.
- Deployment: install, upgrade, rollback, uninstall.

Un test definito ma non eseguito è `NOT RUN`. Il risultato deve essere legato a commit, versione, ambiente ed evidenza.

## 9. Vertical slice MVP

1. Fixture evento.
2. Collector/ingest Go.
3. Normalizzazione.
4. Scrittura SQLite.
5. Regola demo.
6. Detection e incidente.
7. IPC.
8. Dashboard.
9. Restart e verifica integrità.

Niente auto-response. La prima milestone è dimostrare un flusso completo e ripetibile, non implementare subito ogni feature.

## 10. Roadmap

### Fase A — Fondazioni
Repository, CI, decisioni ADR, logging, config, schema DB, service lifecycle.

### Fase B — MVP end-to-end
IPC minimo, collector, normalizer, storage, detection demo, incident, UI.

### Fase C — V1 robusta
Dedup, correlazione, inventario, vulnerability checks, backup/restore, audit, diagnostics, installer.

### Fase D — V1 estesa
Telegram opt-in, simulation, miglioramenti detection, release pipeline e SBOM.

### Fase E — V2+
Linux/macOS, agenti remoti, identità, enrollment/revoca, sync e coordinamento.

## 11. Test di release obbligatori

- Test P0 superati.
- Security test applicabili superati.
- Installazione su OS supportati.
- Update e rollback verificati.
- Backup e restore dimostrati.
- Nessun segreto in repo/log.
- Dipendenze scansionate.
- SBOM generata.
- Known issues documentati.
- Matrice tracciabilità aggiornata.
- Nessun bug critico aperto senza accettazione del rischio.

## 12. Decisioni ancora aperte

- Named Pipes e modello ACL IPC.
- Versioni Windows minime supportate.
- Schema SQL definitivo e gestione dedup/null.
- Config format e precedence.
- Relazioni approvals/actions.
- Retention e budget storage.
- Protezione/cifratura backup.
- Budget performance su macchina di riferimento.
- Fonti eventi Windows da includere nell'MVP.
- Confini FFI/IPC e packaging dei moduli Rust.
- Isolamento, packaging e gestione errori dei worker Python.

## 13. Proposte aggiuntive utili

Le seguenti sono proposte, non requisiti già approvati:

1. **Modalità demo con dataset sintetico:** permette di esplorare dashboard e detection senza raccogliere dati reali.
2. **Health score per copertura, non per “sicurezza assoluta”:** indicare fonti attive, ultime acquisizioni, controlli mancanti e qualità dei dati; evitare un punteggio che suggerisca protezione garantita.
3. **Explainability per ogni alert:** “perché lo vedo”, dati usati, regola/versione e cosa non è stato verificato.
4. **Registro delle modifiche (change journal):** timeline di configurazioni, aggiornamenti, regole e restore.
5. **Export/import regole firmate:** utile per distribuire set di regole senza modificare il core, con firma, versione e test.
6. **Test di backup automatico su copia isolata:** non basta creare il backup; verificare periodicamente che sia ripristinabile.
7. **Budget risorse configurabile:** limiti a CPU, RAM, dimensione log e spazio DB, con comportamento degradato sicuro.
8. **Privacy dashboard:** mostra cosa resta locale, cosa viene esportato e quali integrazioni sono attive.
9. **Compatibilità router trasparente:** catalogare solo fonti/integrazioni testate, senza promettere visibilità universale.
10. **Canary release e rollback:** riduce il rischio di aggiornamenti difettosi.
11. **Dataset di regressione versionato:** consente di misurare se una modifica migliora o peggiora i falsi positivi.
12. **Accessibilità UI:** navigazione tastiera, contrasto, descrizioni testuali e messaggi non basati solo sul colore.

## 14. Rischi principali

- Visibilità limitata su IoT/router consumer.
- Falsi positivi e falsi negativi.
- Privilegi eccessivi per fonti OS.
- Complessità di update e migrazioni.
- Backup apparentemente valido ma non ripristinabile.
- Aumento della superficie per runtime multipli.
- Dipendenza da servizi esterni per notifiche.

Mitigare con scope realistico, test riproducibili, secure defaults e documentazione dei limiti.

## 15. Regola conclusiva

La qualità HSOC non si misura dal numero di feature, ma dalla correttezza del flusso dati, dalla sicurezza dei confini, dalla spiegabilità dei rilevamenti, dalla capacità di recupero e dalle prove verificabili. Ogni release deve dimostrare ciò che dichiara.

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

### Decisioni definitive aggiunte nella revisione 1.1

- Licenza MIT.
- Monorepo GitHub.
- Dashboard Tauri + React + TypeScript.
- Stack Go + Rust + Python: Go core e unico writer SQLite; Rust sicurezza specialistica; Python worker/analisi ausiliarie.
- Docker facoltativo, solo sviluppo/test.
- Agenti remoti implementati dalla V2, non nella V1.
- Documento 16 dedicato a GitHub, CI/CD e DevSecOps.
