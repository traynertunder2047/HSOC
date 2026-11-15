# HSOC — Database Specification

**Versione:** 1.1-draft | **Database:** SQLite locale

## 1. Principi

- SQLite è il database primario locale della V1.
- Solo il servizio Go può scrivere o migrare il database.
- UI, worker Python/Rust e processi futuri usano API/IPC.
- Foreign key abilitate; transazioni per operazioni multi-tabella.
- Migrazioni versionate, atomiche e testate.
- Segreti non memorizzati in chiaro nel DB: usare Credential Manager/OS secret store.
- Timestamp in UTC, visualizzazione locale in UI.
- WAL può essere adottato se compatibile con il piano di backup; il backup deve essere WAL-aware.

## 2. Entità logiche

| Entità | Scopo | Campi indicativi |
|---|---|---|
| `schema_migrations` | Versione schema | version, applied_at, checksum |
| `events` | Eventi normalizzati | id, source, event_type, occurred_at, received_at, payload, fingerprint |
| `detections` | Risultati regole | id, event_id, rule_id, rule_version, severity, confidence, rationale |
| `incidents` | Caso aggregato | id, title, status, severity, opened_at, updated_at |
| `incident_events` | Collegamento molti-a-molti | incident_id, event_id |
| `rules` | Metadati regole | rule_id, version, enabled, definition, updated_at |
| `inventory_items` | Software e componenti | id, name, version, publisher, source, observed_at |
| `vulnerability_findings` | Risultati controllo | id, component_id, advisory_id, severity, status, detected_at |
| `actions` | Azioni proposte/eseguite | id, incident_id, action_type, status, created_at |
| `approvals` | Approvazioni | id, action_id, decision, decided_at, actor |
| `audit_log` | Traccia operazioni | id, actor, operation, target, result, timestamp |
| `config_snapshots` | Versioni configurazione | id, version, content_hash, created_at |
| `backup_records` | Inventario backup | id, path, created_at, checksum, status |
| `health_snapshots` | Stato componenti | id, component, status, detail, observed_at |

Lo schema fisico definitivo (tipi, indici, CHECK, FK, nullability e migrazioni) deve essere fissato in ADR/schema review prima del codice persistente.

## 3. Vincoli e indici

- Chiavi primarie stabili; preferire UUID/ULID o ID generati dal servizio.
- `events.fingerprint` indicizzato per dedup.
- Indici su timestamp, severity, status e FK usate nelle query UI.
- CHECK per enum e intervalli numerici.
- FK con comportamento `RESTRICT`/`CASCADE` deciso per ogni relazione; non usare cascade distruttivi senza motivazione.
- `incident_events` con unique composite `(incident_id, event_id)`.
- Regole e finding conservano versione e provenienza.
- Payload con dimensione massima e schema esplicito.

## 4. Deduplicazione

La dedup non deve cancellare la prova originale. Definire:
1. fingerprint deterministico su campi normalizzati;
2. finestra temporale configurabile;
3. trattamento dei campi null;
4. comportamento in caso di collisione;
5. collegamento tra occorrenze ripetute e record aggregato.

**Decisione aperta:** vincoli UNIQUE con colonne nullable possono avere semantiche non intuitive in SQLite. Valutare fingerprint non-null calcolato dal servizio o indice univoco parziale.

## 5. Integrità e concorrenza

- `PRAGMA foreign_keys=ON` per ogni connessione.
- Transazioni esplicite per event → detection → incident.
- Retry limitato per `SQLITE_BUSY`; non loop infiniti.
- Timeout e pool connessioni definiti.
- `integrity_check` e `foreign_key_check` in diagnostica/manutenzione.
- Gestire disk full, file read-only, permessi errati e corruzione come errori espliciti.
- UI non deve aprire il file DB né copiarlo mentre è attivo.

## 6. Retention e privacy

- Retention configurabile con default conservativo documentato.
- Eliminazione in batch e audit dell'operazione.
- Separare retention eventi, incidenti, audit e backup.
- Non registrare token, password, chiavi o dati personali non necessari.
- Export diagnostico con redazione dei segreti e minimizzazione dei dati.

## 7. Backup e restore

- Backup eseguito dal servizio con API SQLite coerente o procedura che includa correttamente WAL.
- Manifest con versione schema, timestamp UTC, checksum e versione HSOC.
- Backup cifrato quando contiene dati sensibili e la chiave è gestita fuori dal backup.
- Verifica checksum prima del restore.
- Restore preferibilmente su file temporaneo; verifica integrità; sostituzione atomica quando possibile.
- Conservare almeno una copia precedente fino alla verifica del ripristino.
- Non sovrascrivere il DB attivo senza conferma e procedura di recovery.

## 8. Migrazioni

Ogni migrazione deve:
- avere ID e checksum;
- essere transazionale quando supportato;
- essere testata da N-1 a N;
- gestire backup pre-migrazione;
- essere idempotente o impedire chiaramente la doppia esecuzione;
- non perdere dati silenziosamente;
- avere piano di rollback o dichiarare perché il rollback richiede restore.

## 9. Questioni aperte

- Struttura `policy_assignments`.
- Relazione circolare tra `approvals` e `actions`.
- Strategia di re-enrollment per futuri agenti remoti.
- Chiavi e indici definitivi per dedup.
- Retention predefinita e limiti di spazio.
- Cifratura DB/backup e compatibilità con costo zero.

## 10. Test database

| ID | Procedura sintetica | Risultato atteso |
|---|---|---|
| DB-001 | Aprire DB nuovo | Schema creato alla versione prevista |
| DB-002 | Eseguire migrazione N-1 → N | Migrazione applicata una volta |
| DB-003 | Ripetere migrazione già applicata | Nessuna doppia modifica |
| DB-004 | Inserire FK inesistente | Operazione rifiutata |
| DB-005 | Inserire event duplicato | Dedup secondo regola documentata |
| DB-006 | Simulare errore a metà transazione | Rollback completo |
| DB-007 | DB read-only | Errore gestito, servizio non corrompe dati |
| DB-008 | Disk full simulato | Errore visibile, integrità verificata |
| DB-009 | Backup con WAL attivo | Backup coerente e verificabile |
| DB-010 | Restore su DB di test | Integrità e conteggi coerenti |
| DB-011 | Retention scaduta | Solo record eleggibili rimossi |
| DB-012 | Audit retention | Operazione registrata senza segreti |
| DB-013 | `integrity_check` | Esito OK sul DB valido |
| DB-014 | Concorrenza letture/scritture | Nessuna perdita o corruzione |

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

### Confini database rispetto allo stack

- Il servizio Go è l'unico writer SQLite e gestisce migrazioni, transazioni, backup e restore.
- I moduli Rust e i worker Python ricevono i dati necessari tramite interfacce interne controllate e restituiscono risultati strutturati; non aprono il file SQLite.
- La dashboard Tauri/React/TypeScript usa esclusivamente IPC documentato verso il servizio Go.
- I futuri agenti remoti V2 non ricevono accesso diretto al database locale: useranno API/protocolli autenticati da progettare nella fase V2.
