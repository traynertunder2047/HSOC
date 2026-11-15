# HSOC — Testing & Quality Assurance

**Versione:** 1.1-draft | **Stato:** piano di test, non report di esecuzione

## 1. Obiettivo

Dimostrare con prove ripetibili che ogni requisito implementato funziona, che i regressions sono intercettati e che il software gestisce errori e recovery in modo sicuro.

## 2. Livelli di test

- **Unit:** funzioni, parser, normalizer, scoring, config, repository.
- **Integration:** DB, IPC, migrazioni, notifiche mock, backup.
- **System:** installazione, servizio, UI, flussi end-to-end.
- **Security:** ACL, autorizzazione, input validation, segreti, update.
- **Reliability:** crash, restart, disk full, DB busy, backup/restore.
- **Performance:** CPU, RAM, I/O, crescita DB su carico sintetico.
- **Detection quality:** precision, recall, falsi positivi e falsi negativi su dataset etichettato.
- **Compatibility:** OS supportati e combinazioni UI/service.
- **Acceptance:** criteri MVP e release gate.

## 3. Test case obbligatorio

Ogni test deve avere:
- ID stabile;
- requisito `TR-*` collegato;
- titolo e obiettivo;
- versione/commit;
- prerequisiti e ambiente;
- fixture/input;
- procedura riproducibile;
- risultato atteso;
- risultato effettivo;
- evidenza (log, screenshot, report);
- stato: `NOT RUN`, `PASS`, `FAIL`, `BLOCKED`, `SKIPPED`;
- autore e data.

## 4. Catalogo minimo

### Core e integrazione
- INT-01: avvio servizio e health.
- INT-02: collector → normalizer.
- INT-03: normalizer → storage.
- INT-04: detection → incident.
- INT-05: IPC → UI.
- INT-06: config reload valid.
- INT-07: config reload invalid.
- INT-08: audit operazione sensibile.
- INT-09: worker opzionale assente.
- INT-10: recupero dopo disconnessione UI.

### Detection
- DET-01..05: regole positive con fixture note.
- DET-06..08: eventi negativi simili ma benigni.
- DET-09: dedup.
- DET-10: correlazione.
- DET-11: severity.
- DET-12: confidence.
- DET-13: versione regola.
- DET-14: evento incompleto/malformato.
- DET-15: regressione su dataset storico sintetico.

### Configurazione
- CFG-01..05: default e schema.
- CFG-06..08: precedence e override.
- CFG-09: policy obbligatoria non disattivabile.
- CFG-10: reload atomico.
- CFG-11: rollback config.
- CFG-12: segreti in credential store.
- CFG-13: config backup/restore.
- CFG-14: input invalido.
- CFG-15: audit modifiche.

### Deployment
- DEP-01..14: fare riferimento al Documento 05; mantenere evidenza per ogni OS/build testata.

### Recovery
- REC-01: backup completo.
- REC-02: checksum errato.
- REC-03: restore su DB di test.
- REC-04: restore interrotto.
- REC-05: recovery dopo crash.
- REC-06: migrazione fallita.
- REC-07: DB read-only.
- REC-08: spazio insufficiente.

## 5. Metriche detection

Per categoria e soglia:
- TP: veri positivi;
- FP: falsi positivi;
- TN: veri negativi;
- FN: falsi negativi;
- Precision = TP / (TP + FP);
- Recall = TP / (TP + FN);
- F1 = media armonica di precision e recall.

Riportare supporto del dataset, periodo, versione regole, soglia e limiti. Non usare un unico punteggio aggregato per nascondere categorie deboli. Non ottimizzare soltanto la riduzione degli alert: un sistema che non segnala nulla può avere pochi FP ma anche molti FN.

## 6. Criteri di qualità

- Nessun crash non gestito nei test di regressione.
- Nessun difetto critico/alto aperto senza accettazione formale del rischio.
- Backup e restore superati.
- Tutti i test P0 passati o bloccati con motivazione approvata.
- Nessun segreto rilevato dai secret scanner.
- Nessuna vulnerabilità critica non mitigata nelle dipendenze.
- Performance entro budget definiti prima del benchmark.
- Ogni requisito P0 ha almeno un test associato.

## 7. Test performance da definire

Misurare baseline su hardware di riferimento dichiarato:
- idle CPU/RAM del servizio;
- picco CPU durante ingest;
- throughput eventi sintetici;
- latenza ingest-to-dashboard;
- dimensione DB per giorno/volume;
- tempo backup/restore;
- comportamento con DB grande.

Non fissare soglie arbitrarie prima di avere una baseline realistica.

## 8. CI consigliata

- Go: `gofmt`, `go test ./...`, `go vet ./...`, `govulncheck`.
- Go: `gofmt` check, `go vet`, `go test ./...`, `govulncheck`.
- Rust: `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo test`, `cargo audit`.
- Python: `ruff check`, `ruff format --check`, `pytest`, `pip-audit`.
- Dashboard Tauri/React/TypeScript: lint, typecheck, test e build.
- Tutti i controlli si applicano ai componenti effettivamente presenti nel repository; il piano prevede workflow separati per ciascuno.
- UI: typecheck, lint, test e build.
- Secret scanning: Gitleaks.
- Vulnerability/dependency inventory: OSV-Scanner o strumento equivalente.
- Test migrazioni e backup in pipeline isolata.

## 9. Gate release

**GO** se: build, test P0, test security, backup/restore, install/update e documentazione sono verificati.  
**NO-GO** se: test critici falliscono, firma update non verificabile, restore non funziona, segreti sono esposti o esistono bug che possono corrompere dati o causare azioni non autorizzate.

## 10. Test report

Non marcare mai un test PASS senza output/evidenza. I test non eseguiti restano NOT RUN. Un test bloccato non equivale a PASS.

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

### Test dei confini multi-linguaggio

Aggiungere test per:
- input malformato e payload oltre il limite tra Go e Rust;
- timeout, crash e riavvio di worker Python;
- schema/versione incompatibile tra componenti;
- risultato incompleto o non valido restituito da un worker;
- conferma che solo Go scriva su SQLite;
- dashboard che comunica solo tramite IPC autorizzato;
- build desktop senza Docker installato.
