# HSOC — Regole obbligatorie di sviluppo

**Versione:** 1.1-draft | **Applicazione:** ogni contributo e modifica

## 1. Regole architetturali

1. Go è l'autorità centrale della V1.
2. Solo Go scrive SQLite.
3. UI comunica con Go via IPC documentato.
4. Nessun modulo opzionale accede direttamente al DB.
5. Lo stack approvato comprende Go, Rust e Python: Go è il core, Rust i moduli specialistici di sicurezza e Python i worker/analisi ausiliarie. L'integrazione avviene per fasi e con contratti testati.
6. Agenti remoti non entrano nella V1.
7. Non esporre porte di rete per default.
8. Non introdurre hardware esterno obbligatorio.

## 2. Regole di sicurezza

1. Minimo privilegio in ogni processo.
2. Ogni input esterno è non fidato.
3. Validare tipo, dimensione, encoding e range.
4. Non eseguire comandi shell derivati da input.
5. Non memorizzare segreti in chiaro in config, DB, log o repository.
6. Usare Credential Store del sistema operativo.
7. Redigere token e dati sensibili nei log.
8. Default Telegram OFF.
9. Default auto-response OFF.
10. Operazioni sensibili autorizzate, confermate e auditate.
11. Nessun aggiornamento non verificato.
12. Fail-safe: in caso di errore, non applicare azioni rischiose.

## 3. Regole Go

- Formattare con `gofmt`.
- Usare error wrapping e contesto.
- Gestire context cancellation e timeout.
- Evitare goroutine senza chiusura deterministica.
- Non usare global state mutabile senza motivazione.
- Repository layer unico per SQLite.
- Transazioni per modifiche multi-entità.
- Query parametrizzate.
- Limiti a payload, batch e concorrenza.
- Test unitari per logica e failure path.

## 4. Regole Rust

Applicabili a tutti i moduli Rust del progetto:
- `cargo fmt --check`, `cargo clippy -- -D warnings` e `cargo test`;
- gestione esplicita errori;
- confine FFI/IPC ristretto e versionato;
- nessun accesso DB;
- nessun privilegio aggiuntivo implicito;
- `cargo audit` e inventory dipendenze;
- test su input malformato e panic boundary.

## 5. Regole Python

Applicabili a tutti i worker e moduli Python del progetto:
- ambiente isolato e lockfile;
- `ruff`, `pytest` e `pip-audit`;
- nessun accesso diretto al DB;
- niente segreti in notebook/log;
- niente esecuzione automatica di script generati;
- input/output schema-validati;
- Python non deve impedire il funzionamento del core.

## 6. Regole UI

- Nessuna query diretta a SQLite.
- Nessuna fiducia implicita nei dati visualizzati.
- Escaping dei contenuti e prevenzione injection.
- Gestione stati loading/error/empty/offline.
- Conferma per operazioni distruttive.
- Mostrare severità separata da confidence.
- Spiegare fonti, limiti e motivazioni.
- Nessun segreto in local storage o log UI.

## 7. Regole database

- Migrazioni numerate, versionate e testate.
- FK abilitate.
- Query parametrizzate.
- Backup WAL-aware.
- Nessuna cancellazione silenziosa.
- Retention documentata e auditata.
- Test integrità dopo restore.
- DB file e directory con ACL minime.

## 8. Regole detection

- Ogni regola ha ID e versione.
- Ogni regola ha fixture positive e negative.
- Motivazione leggibile per ogni detection.
- Confidence e severity distinte.
- Feedback false positive non cancella evidenza.
- Ogni modifica a una regola esegue regressione.
- Nessuna risposta automatica in MVP.
- Metriche per categoria, non solo aggregate.

## 9. Regole di processo

Ogni modifica segue:
1. requisito/bug e ID;
2. analisi impatto e threat review;
3. design/ADR se necessario;
4. implementazione piccola;
5. unit test;
6. integration/security test applicabili;
7. review;
8. aggiornamento documentazione e matrice;
9. merge solo con CI verde;
10. evidenza associata alla release.

## 10. Regole Git e review

- Branch e commit descrittivi.
- PR piccole e focalizzate.
- Nessun segreto o dato utente nel repository.
- Nessun binario generato committato senza ragione.
- Almeno una review per modifiche a sicurezza, storage, IPC o response.
- Ogni eccezione alle regole deve essere motivata e registrata.

## 11. Regole per dipendenze

- Aggiungere una dipendenza solo con motivazione.
- Preferire librerie mantenute e con licenza compatibile.
- Bloccare versioni tramite lockfile.
- Eseguire scan vulnerabilità.
- Rimuovere dipendenze non usate.
- Non auto-aggiornare in produzione senza test.

## 12. Checklist PR

- [ ] Requisito/bug collegato.
- [ ] Threat/permission impact valutato.
- [ ] Nessun accesso DB fuori da Go storage.
- [ ] Nessun segreto introdotto.
- [ ] Error handling e logging redatto.
- [ ] Unit test aggiunti/aggiornati.
- [ ] Integration/security test eseguiti quando richiesto.
- [ ] Test precedenti non regressi.
- [ ] Documenti e matrice aggiornati.
- [ ] CI verde.
- [ ] Limiti e decisioni aperte registrati.

## 13. Violazioni bloccanti

Bloccare merge/release se la modifica:
- espone segreti;
- permette accesso IPC non autorizzato;
- esegue comandi arbitrari;
- amplia privilegi senza giustificazione;
- abilita auto-response non approvata;
- bypassa validazione;
- scrive DB da UI/worker;
- elimina dati senza audit/consenso;
- distribuisce update non verificati.

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

### Regole repository e licenza

- Il progetto usa la licenza MIT, conservata nel file radice `LICENSE`.
- Ogni nuova dipendenza deve avere licenza verificata e, se necessario, attribuzione in `THIRD_PARTY_NOTICES.md`.
- Il repository è organizzato come monorepo.
- La dashboard usa Tauri + React + TypeScript.
- Docker è ammesso per sviluppo/test, ma non può diventare prerequisito per l'utente finale.
- Workflow GitHub Actions con permessi minimi; nessun segreto esposto a workflow di Pull Request non fidati.
- `main` è protetto da Pull Request e controlli CI obbligatori, secondo le funzionalità disponibili.
- I contratti V2 per agenti remoti possono essere documentati, ma listener e agenti remoti non sono implementati nella V1.
