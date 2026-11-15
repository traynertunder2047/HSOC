# HSOC — Consolidamento & Implementation Readiness

**Versione:** 1.1-draft | **Stato:** gate progettuale prima dell'implementazione

## 1. Obiettivo

Verificare che le specifiche siano abbastanza coerenti da avviare lo sviluppo senza ambiguità critiche. Questo documento non dichiara che il codice sia già stato realizzato.

## 2. Principi obbligatori

- Local-first e costo minimo.
- Nessun hardware esterno obbligatorio.
- Go autorità centrale.
- SQLite scritto solo da Go.
- UI separata dal servizio.
- Minimo privilegio.
- Privacy e minimizzazione dati.
- Config validata e versionata.
- Backup/restore testabili.
- Tracciabilità requisiti-test.
- Agenti remoti esclusi dalla V1.
- Auto-response disattivata.
- IoT osservato passivamente.

## 3. Decisioni da consolidare

| ADR | Tema | Proposta |
|---|---|---|
| ADR-121 | IPC | Named Pipes Windows con ACL; validare identità client |
| ADR-122 | Rust | Linguaggio approvato per moduli specialistici di sicurezza; definire confini FFI e packaging |
| ADR-123 | Python | Linguaggio approvato per worker e analisi ausiliarie; definire isolamento e packaging |
| ADR-124 | Auto-response | OFF nella V1; approvazione esplicita |
| ADR-125 | Config | Schema versionato e precedence definita |
| ADR-126 | Fonte di verità | DB per stato runtime; config per bootstrap/preferenze |
| ADR-127 | Health monitor | Stato per componente e degradazione |
| ADR-128 | Diagnostics | Export volontario e redatto |
| ADR-129 | Simulation | Modalità senza effetti sul sistema |
| ADR-130 | Agenti remoti | Esclusi dalla V1 |
| ADR-131 | Architettura | Windows x86-64 iniziale |
| ADR-132 | Windows supportati | Definire e testare matrice OS |

## 4. Definition of Ready

Una feature è pronta per sviluppo se:
- requisito e motivazione sono definiti;
- versione/priorità sono assegnate;
- comportamento normale e failure mode sono descritti;
- dati trattati e privacy sono chiari;
- minacce e autorizzazioni sono valutate;
- interfacce e schema sono definiti;
- test positivi, negativi e recovery sono scritti;
- dipendenze e limiti sono noti;
- documenti e matrice tracciabilità sono aggiornati.

## 5. Definition of Done

Una feature è completata solo quando:
- implementazione e review sono concluse;
- unit/integration test passano;
- test security applicabili passano;
- log/error handling sono adeguati;
- nessun segreto è esposto;
- documentazione aggiornata;
- requisito collegato a test ed evidenza;
- limiti noti registrati;
- feature flag/default sicuro impostato;
- build riproducibile.

## 6. Primo vertical slice

Evento sintetico/OS non distruttivo → collector Go → validazione/normalizzazione → SQLite → regola demo → detection → incident → IPC → dashboard.

Criteri:
- fixture riproducibile;
- un solo percorso end-to-end;
- niente azioni automatiche;
- DB integro dopo restart;
- UI non accede al DB;
- test automatici e report allegati.

## 7. Piano test per gate

- Fondamenta: unit test, config, logging, lifecycle.
- Storage: migrazioni, FK, transazioni, dedup.
- IPC: autorizzazione, schema, timeout, rate.
- Detection: fixture positive/negative, metriche.
- UI: stati, errori, accessibilità.
- Recovery: backup, restore, crash, disk full.
- Security: ACL, segreti, update, least privilege.
- Release: installer, upgrade, rollback, SBOM.

## 8. Sequenza di sviluppo

1. Repository, standard, CI e documentazione.
2. Modello dati e migrazioni.
3. Service lifecycle/health.
4. IPC.
5. Collector/normalizer.
6. Detection/incident.
7. Dashboard.
8. Backup/restore.
9. Inventory/vulnerability checks.
10. Notifications opt-in.
11. Hardening e installer.
12. Release candidate.

## 9. Rischi bloccanti

- IPC senza autorizzazione robusta.
- DB scrivibile da più componenti.
- Nessun restore testato.
- Servizio con privilegi eccessivi.
- Update non verificato.
- Azioni automatiche non approvate.
- Telemetria IoT sovrastimata.
- Requisiti P0 senza test.

## 10. GO/NO-GO

**GO:** architettura coerente, decisioni P0 chiuse, vertical slice ripetibile, test P0 passati, recovery verificato.  
**NO-GO:** perdita dati, accesso non autorizzato, segreti esposti, azioni non autorizzate, restore fallito o test critici mancanti.

## 11. Primo sprint raccomandato

- Creare repository e CI.
- Congelare schema evento canonico e primo schema DB.
- Prototipare service lifecycle e health.
- Definire e testare IPC minimo.
- Implementare vertical slice con fixture.
- Produrre primo report test e aggiornare matrice.

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

### Gate di prontezza aggiornato

La scelta dello stack Go + Rust + Python e della dashboard Tauri + React + TypeScript è confermata. Prima di iniziare ciascun modulo vanno comunque definiti contratto, input/output, failure mode, dipendenze, packaging e test. Il fatto che un linguaggio sia approvato non autorizza un modulo privo di requisito o di responsabilità chiara.

Docker resta facoltativo per sviluppo/test. La V1 resta locale e Windows-first; agenti remoti soltanto dalla V2.
