# HSOC — Detection & Response Specification

**Versione:** 1.1-draft | **Modalità:** risposta ibrida conservativa

## 1. Pipeline

```text
Fonte OS
  -> Collector
  -> Validazione
  -> Normalizzazione
  -> Persistenza evento
  -> Regole
  -> Detection
  -> Dedup/Correlazione
  -> Incident
  -> Dashboard/Notifica
  -> Azione proposta (simulazione/approvazione)
```

Ogni passaggio deve avere error handling e log strutturato.

## 2. Evento canonico

Campi logici:
- `event_id`
- `source`
- `event_type`
- `occurred_at`
- `received_at`
- `host_id` locale
- `severity_hint` opzionale
- `attributes` validati
- `raw_reference` o payload redatto
- `schema_version`

I dati grezzi sono non fidati. Limitare dimensione e campi conservati.

## 3. Regole

Ogni regola include:
- ID stabile;
- versione;
- descrizione;
- categoria;
- condizioni;
- severità;
- confidence;
- fonti necessarie;
- spiegazione del match;
- eventuali eccezioni;
- test positivi e negativi;
- stato enabled/disabled.

Le regole devono essere deterministiche per lo stesso input e versione. Evitare regole opache senza motivazione.

## 4. Severità e confidence

Severità misura l'impatto potenziale; confidence misura quanto il segnale supporta la conclusione. Non confondere i due valori. Mostrare entrambi e spiegare come sono stati assegnati.

## 5. Falsi positivi e negativi

- Ogni regola deve avere fixture positive e negative.
- L'utente può contrassegnare un detection come falso positivo con motivazione.
- La correzione regola richiede nuova versione e regressione.
- Non sopprimere globalmente eventi simili senza analisi.
- Monitorare precision/recall per categoria.
- Non promettere zero falsi positivi.

## 6. Dedup e correlazione

- Dedup su fingerprint e finestra documentata.
- Conservare conteggio e prima/ultima occorrenza.
- Correlare solo eventi con relazione esplicita e testabile.
- Non unire incidenti solo per somiglianza vaga.
- Conservare collegamenti evento-detection-incident.

## 7. Incident lifecycle

Stati: `OPEN`, `ACKNOWLEDGED`, `INVESTIGATING`, `RESOLVED`, `FALSE_POSITIVE`.  
Ogni transizione registra actor, timestamp, stato precedente, stato nuovo e motivazione.

## 8. Risposta ibrida

V1:
- Nessuna azione automatica distruttiva.
- Suggerimenti e simulazione.
- Azioni sensibili richiedono conferma esplicita.
- Azioni devono essere circoscritte, reversibili ove possibile, auditabili.
- Non eseguire comandi arbitrari.
- Se confidence bassa o dati insufficienti, non agire automaticamente.

Possibili azioni future, da valutare una per una: disabilitare temporaneamente una regola locale, mettere in quarantena un elemento solo con integrazione OS sicura, o guidare l'utente verso una procedura manuale. Nessuna è abilitata per default.

## 9. IoT

La V1 può esporre inventario osservato passivamente e informazioni disponibili dal sistema/rete. Non eseguire scansioni intrusive. Segnalare esplicitamente:
- fonte del dato;
- ultima osservazione;
- livello di completezza;
- router/protocolli non supportati;
- incertezza sull'identificazione.

## 10. Test detection

| ID | Scenario | Risultato atteso |
|---|---|---|
| DET-01 | Fixture che soddisfa regola | Detection atteso |
| DET-02 | Fixture benigno simile | Nessun detection non giustificato |
| DET-03 | Evento incompleto | Gestione controllata |
| DET-04 | Evento duplicato | Dedup secondo policy |
| DET-05 | Eventi correlati | Un incidente con riferimenti corretti |
| DET-06 | Eventi non correlati | Incidenti separati |
| DET-07 | Regola disabilitata | Nessuna detection da quella regola |
| DET-08 | Versione regola aggiornata | Versione registrata nel risultato |
| DET-09 | Confidence bassa | Nessuna azione automatica |
| DET-10 | False positive feedback | Audit e stato aggiornato |
| DET-11 | Simulazione risposta | Nessun cambiamento di sistema |
| DET-12 | Azione sensibile senza approvazione | Bloccata |
| DET-13 | Azione approvata | Solo azione consentita e auditata |
| DET-14 | Restart durante pipeline | Nessuna perdita/corruzione non gestita |
| DET-15 | Dataset regressione | Metriche calcolate e confrontabili |

## 11. Metriche

Precision, recall, F1, FP per 1.000 eventi, tempo ingest-to-detection, percentuale eventi non classificabili e coverage per fonte. Riportare limiti dataset e versioni.

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

### Responsabilità dei linguaggi nella pipeline detection

Go mantiene pipeline, stato, regole attive, persistenza e audit. Rust può fornire funzioni specialistiche di sicurezza attraverso interfacce ristrette. Python può eseguire analisi ausiliarie e worker non autorevoli. Nessun risultato prodotto da Rust/Python viene considerato attendibile senza validazione Go. I worker non applicano direttamente azioni al sistema e non scrivono SQLite.
