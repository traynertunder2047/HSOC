# HSOC — Deployment & Installation

**Versione:** 1.1-draft | **Target iniziale:** Windows x86-64

## 1. Obiettivo

Definire un'installazione ripetibile del servizio Go e della dashboard desktop. La versione iniziale non richiede hardware esterno né account cloud.

## 2. Piattaforma

- Prima piattaforma: Windows x86-64.
- Versioni Windows supportate: **DA CONFERMARE** con matrice testata.
- Linux/macOS: V2+ dopo portabilità del core.
- Non dichiarare supporto per versioni OS non testate.

## 3. Componenti distribuiti

- Servizio HSOC Go.
- Dashboard Tauri.
- Configurazione iniziale.
- Schema/migrazioni SQLite.
- Regole di rilevamento versionate.
- Documentazione, licenze e SBOM.
- Updater solo quando firma e rollback sono pronti.

## 4. Directory logiche

I path definitivi devono rispettare convenzioni Windows e ACL:
- binari installati in directory protetta;
- configurazione in area dedicata;
- database e dati runtime in ProgramData o directory utente definita;
- log separati dai segreti;
- backup in destinazione scelta dall'utente;
- cache temporanee separata dai dati persistenti.

Non hardcodare path assoluti; usare API OS e testare ACL.

## 5. Installazione

1. Verificare firma e integrità del pacchetto.
2. Controllare OS e architettura.
3. Creare directory con ACL minime.
4. Installare binari e file di policy.
5. Inizializzare DB e migrazioni.
6. Registrare il servizio con account/privilegi minimi.
7. Avviare servizio e controllare health.
8. Avviare UI e verificare IPC.
9. Mostrare stato iniziale e limiti della copertura.
10. Non abilitare Telegram o auto-response senza opt-in.

## 6. Aggiornamento

- Pacchetto firmato e verificato.
- Backup pre-update.
- Controllo spazio e compatibilità.
- Migrazione DB controllata.
- Verifica health post-update.
- Rollback del binario e restore DB se necessario.
- Nessun update silenzioso che cambia policy sensibili.

## 7. Disinstallazione

- Arresto servizio.
- Rimozione registrazione servizio e binari.
- Dati utente conservati per default, con opzione esplicita per rimuoverli.
- Prima di cancellare DB/backup, mostrare conseguenze e chiedere conferma.
- Rimuovere credenziali dal Credential Store.
- Registrare esito; non lasciare processi orfani.

## 8. Recovery

- Se servizio non parte, UI mostra istruzioni e correlation ID.
- Se DB non valido, non sovrascriverlo automaticamente.
- Proporre ripristino da backup verificato.
- Conservare copia del DB problematico solo se l'utente lo autorizza e con protezione adeguata.

## 9. Test deployment

| ID | Test | Risultato atteso |
|---|---|---|
| DEP-01 | Installazione pulita | Servizio e UI installati |
| DEP-02 | OS non supportato | Installazione bloccata con messaggio |
| DEP-03 | Firma pacchetto non valida | Installazione bloccata |
| DEP-04 | ACL directory | Accesso limitato |
| DEP-05 | Primo avvio | DB inizializzato, health OK |
| DEP-06 | Riavvio Windows | Servizio recupera secondo policy |
| DEP-07 | UI chiusa | Servizio continua senza crash |
| DEP-08 | Aggiornamento N-1 → N | Dati preservati e health OK |
| DEP-09 | Update interrotto | Recovery/rollback documentato |
| DEP-10 | Spazio disco insufficiente | Errore gestito |
| DEP-11 | Disinstallazione conserva dati | Dati presenti e servizio rimosso |
| DEP-12 | Disinstallazione con rimozione dati | Solo dopo conferma |
| DEP-13 | Reinstallazione con DB esistente | Nessuna sovrascrittura non autorizzata |
| DEP-14 | Installazione senza hardware aggiuntivo | Funzioni core operative |

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

### Packaging e runtime dei linguaggi

La distribuzione Windows deve includere i componenti runtime necessari per l'utente senza richiedere installazioni separate di Go, Rust, Python, Node.js o Docker. Il packaging dovrà incorporare l'eseguibile Go, i moduli Rust compilati e i worker Python con il runtime/deployment scelto e le relative dipendenze bloccate. La dashboard Tauri distribuisce gli asset React/TypeScript compilati.

La modalità concreta di packaging dei worker Python e l'eventuale uso di processi separati sono decisioni tecniche da validare con prototipo. Docker non è incluso come requisito per l'installazione dell'utente.
