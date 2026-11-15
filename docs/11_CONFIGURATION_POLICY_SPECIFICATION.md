# HSOC — Configuration & Policy Specification

**Versione:** 1.1-draft

## 1. Scopo

Separare preferenze operative, configurazione tecnica, stato runtime e segreti. Una singola fonte di verità deve essere definita per ogni categoria di dato.

## 2. Categorie

- **Config file:** impostazioni statiche e bootstrap, schema versionato.
- **Database:** stato runtime, eventi, incidenti, audit, inventario, stato backup.
- **OS Credential Store:** token e segreti.
- **Policy:** limiti di sicurezza e autorizzazioni.
- **Environment/build:** valori non segreti necessari a packaging/test.

Non duplicare la stessa impostazione in più fonti senza regola di precedence.

## 3. Precedence proposta

1. Vincoli di sicurezza incorporati e non disattivabili.
2. Policy amministrative locali documentate.
3. Configurazione locale valida.
4. Default di prodotto.

Ordine finale da confermare con ADR-125. Nessuna variabile ambiente o argomento CLI deve poter disabilitare silenziosamente i controlli di sicurezza.

## 4. Schema

Ogni campo deve avere:
- nome e tipo;
- default;
- range;
- valore ammesso;
- descrizione;
- rischio;
- versione schema;
- effetto del reload;
- eventuale restart richiesto.

## 5. Default sicuri

- Telegram: OFF.
- Risposta automatica: OFF.
- Agenti remoti: OFF/non presenti in V1.
- Listener di rete: assente per default.
- Export esterno: OFF.
- Moduli opzionali: OFF se non installati.
- Log: senza segreti.
- Privilegi: minimi.

## 6. Modifiche

- Validare interamente prima dell'applicazione.
- Applicare atomicamente.
- Se invalida, conservare ultima configurazione valida.
- Auditare cambi sensibili.
- Mostrare diff e impatto.
- Supportare rollback.
- Non cambiare policy di risposta senza conferma.

## 7. Feature flags

Le feature flag non sono un controllo di sicurezza. Non devono essere usate per nascondere controlli obbligatori. Ogni flag deve avere owner, stato default, versione di introduzione, criterio di rimozione e test.

## 8. Test configurazione

| ID | Test | Risultato atteso |
|---|---|---|
| CFG-01 | Primo avvio senza config | Default sicuri |
| CFG-02 | Config valida | Caricamento riuscito |
| CFG-03 | Config malformata | Errore chiaro, fallback sicuro |
| CFG-04 | Campo sconosciuto | Policy di compatibilità documentata |
| CFG-05 | Valore fuori range | Rifiuto |
| CFG-06 | Override di vincolo obbligatorio | Rifiuto |
| CFG-07 | Reload config valida | Applicazione atomica |
| CFG-08 | Reload fallita | Ultima config valida conservata |
| CFG-09 | Segreto in config file | Non consentito o migrazione a Credential Store |
| CFG-10 | Telegram default | Disattivato |
| CFG-11 | Auto-response default | Disattivata |
| CFG-12 | Audit cambio policy | Evento audit presente |
| CFG-13 | Backup config | Config ripristinabile |
| CFG-14 | Rollback config | Versione precedente ripristinata |
| CFG-15 | Restart richiesto | UI comunica necessità e stato |

## 9. Questioni aperte

- Fonte di verità runtime per ogni impostazione.
- Formato config (TOML/YAML/JSON) e schema.
- Policy locale e possibili profili.
- Gestione config multi-versione.
- Secret store per ogni piattaforma futura.

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

### Configurazione dei componenti multi-linguaggio

Le configurazioni dei worker Rust/Python devono essere subordinate alla policy e alla configurazione validate dal servizio Go. I worker non possono ridefinire i controlli di sicurezza, abilitare listener remoti o modificare autonomamente le policy di risposta. I parametri specifici di packaging/runtime sono separati dalle preferenze dell'utente e non devono contenere segreti.
