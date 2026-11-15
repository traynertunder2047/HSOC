# HSOC — Master Technical Blueprint

**Versione:** 1.1-draft | **Stato:** Specifica progettuale

## 1. Scopo

HSOC (Home Security Operations Center) è un'applicazione desktop local-first per aiutare una persona a osservare lo stato di sicurezza del proprio computer Windows e, nei limiti dei dati disponibili, alcuni aspetti dei dispositivi IoT presenti nella rete domestica. Non è un prodotto di videosorveglianza, non richiede telecamere o sensori dedicati e non deve richiedere hardware esterno.

Obiettivi: raccolta locale di eventi, inventario software, rilevamento basato su regole, gestione degli incidenti, dashboard, audit, backup/ripristino e notifiche opzionali.

## 2. Principi non negoziabili

1. Accessibilità e costo operativo minimo; nessun servizio cloud a pagamento necessario per il funzionamento base.
2. Local-first: elaborazione e archiviazione primarie sul dispositivo.
3. Nessun login HSOC obbligatorio nella V1; si riutilizzano identità, sessione e permessi del sistema operativo.
4. Minimo privilegio: il servizio opera con i privilegi più bassi compatibili con le funzioni richieste.
5. Secure-by-default: telemetria esterna, Telegram e risposte automatiche disattivati per impostazione predefinita.
6. Trasparenza: ogni rilevamento mostra motivazione, fonte, confidenza e versione della regola.
7. Reversibilità: backup, restore, rollback e azioni reversibili ove possibili.
8. Nessuna promessa di rilevamento totale o assenza di falsi positivi.
9. La V1 è un agente locale singolo; agenti remoti e coordinamento distribuito sono V2+.
10. Osservazione IoT passiva e non intrusiva; la copertura dipende dal router e dalle informazioni accessibili.

## 3. Ambito funzionale

### V1-MVP (P0)
- Servizio HSOC locale in Go.
- Avvio, arresto controllato e health status.
- Raccolta di una prima fonte di eventi Windows supportata.
- Validazione e normalizzazione degli eventi.
- Persistenza SQLite.
- Regola dimostrativa di rilevamento non distruttiva.
- Creazione e consultazione di un incidente.
- Dashboard desktop che mostra eventi, rilevamenti, incidenti e stato del servizio.
- Log tecnici e diagnostica essenziale.

### V1 (P0/P1)
- Inventario software e componenti/dependency inventory.
- Regole versionate e configurabili con limiti.
- Deduplicazione e correlazione di base.
- Backup e ripristino verificati.
- Gestione configurazione e policy.
- Notifiche locali; Telegram solo come integrazione opt-in.
- Controllo vulnerabilità di sistema e dipendenze, mensile e fuori ciclo per criticità.
- Modalità simulazione per provare regole e policy senza modificare il sistema.
- Audit delle operazioni sensibili.

### V2+
- Agenti remoti con identità, enrollment, revoca e protocollo autenticato.
- Linux e macOS dopo stabilizzazione Windows.
- Coordinamento di più nodi e policy distribuite.
- Automazioni di risposta più avanzate, solo dopo analisi del rischio e approvazione esplicita.

## 4. Fuori ambito

- Videosorveglianza, telecamere, microfoni o sensori dedicati.
- Necessità di acquistare hardware.
- Cloud centrale obbligatorio.
- Accesso remoto aperto a Internet.
- Scansioni intrusive o aggressive dei dispositivi IoT.
- Esecuzione di comandi arbitrari ricevuti da notifiche o interfacce esterne.
- Auto-modifica del codice di produzione o auto-fix non revisionati.
- Garanzia di protezione completa contro ogni minaccia.

## 5. Architettura logica

```text
+-----------------------------+
| Desktop UI                  |
| Tauri + React + TypeScript  |
+--------------+--------------+
               | IPC locale
+--------------v--------------+
| HSOC Core Service (Go)      |
| API/IPC, policy, orchestration|
| collection, detection, audit |
+------+-----------+----------+
       |           |
       |           +--------------------+
       v                                v
+--------------+                +---------------+
| SQLite       |                | OS adapters   |
| local state  |                | Windows first |
+--------------+                +---------------+
       |
       +---- backup/restore
       +---- optional notification adapter

Optional isolated workers:
- Rust: moduli specialistici di sicurezza, con interfacce e responsabilità delimitate.
- Python: worker e attività ausiliarie di analisi, con input/output validati.
- Go resta il core; Rust e Python non accedono direttamente a SQLite.
```

Go è l'autorità centrale per orchestrazione, policy, accesso al database e confini IPC. SQLite deve essere scritto esclusivamente dal servizio Go. UI e worker non devono aprire direttamente il database.

## 6. Stack

| Area | Tecnologia proposta | Vincolo |
|---|---|---|
| Core/service | Go | Autorità centrale |
| Desktop shell | Tauri | Separata dal servizio |
| UI | React + TypeScript | Nessun accesso diretto a SQLite |
| Database | SQLite | Scrittura solo da Go |
| IPC Windows | Named Pipes | ACL e autenticazione locale da validare |
| Sicurezza specialistica | Rust | Linguaggio approvato; moduli integrati per fasi |
| Worker/analisi ausiliaria | Python | Linguaggio approvato; moduli integrati per fasi |
| Test Go | `go test`, `go vet`, `govulncheck` | CI per ogni modifica Go |
| Test Rust | `cargo fmt`, `cargo clippy`, `cargo test`, `cargo audit` | CI per ogni modulo Rust |
| Test Python | `ruff`, `pytest`, `pip-audit` | CI per ogni modulo Python |
| UI | TypeScript compiler, lint e test UI | CI |

Versioni minime e supportate di Windows, Go, Tauri, Node e dipendenze devono essere fissate in una matrice di compatibilità prima della release.

## 7. Moduli principali

- `service`: lifecycle, health e orchestrazione.
- `ipc`: endpoint locale, autorizzazione e serializzazione.
- `config`: caricamento, schema, validazione e precedence.
- `collector`: adattatori di raccolta eventi.
- `normalizer`: schema evento canonico.
- `inventory`: inventario software e versioni.
- `detection`: regole, scoring, dedup e correlazione.
- `incidents`: lifecycle e workflow degli incidenti.
- `response`: azioni consentite, approvazioni e simulazione.
- `storage`: repository SQLite, migrazioni e transazioni.
- `backup`: backup coerenti, verifica e restore.
- `notifications`: notifiche locali e Telegram opt-in.
- `audit`: registro delle operazioni sensibili.
- `diagnostics`: health report ed export redatto.
- `ui`: dashboard e impostazioni.

## 8. Modello di fiducia

Il sistema operativo è il confine primario. Il servizio Go è componente privilegiato rispetto alla UI, ma non deve assumere privilegi amministrativi se non indispensabile. La UI è considerata input non fidato. Gli eventi raccolti sono dati non fidati. Le notifiche sono canali informativi, mai canali di comando.

## 9. Qualità e criteri generali di accettazione

- Installazione e disinstallazione ripetibili.
- Nessun accesso DB diretto dalla UI.
- Errori di input gestiti senza crash.
- Database integro dopo arresto controllato e test di recovery.
- Backup ripristinabile su ambiente di test.
- Ogni rilevamento è spiegabile e tracciabile.
- Nessuna azione automatica distruttiva abilitata nel default.
- Documentazione aggiornata insieme alle modifiche.

## 10. Decisioni da confermare

- ADR-121: Named Pipes Windows, ACL e identità del client.
- ADR-122: confini dei moduli Rust specialistici, FFI e modello di isolamento.
- ADR-123: contratti dei worker Python, packaging e modello di isolamento.
- ADR-124: workflow di risposta e approvazione.
- ADR-125: formato e precedence della configurazione.
- ADR-126: fonte di verità runtime (DB/config).
- ADR-130: esclusione agenti remoti dalla V1.
- ADR-131/132: architettura x86-64 e versioni Windows supportate.

## 11. Test di accettazione principali

| ID | Test | Risultato atteso |
|---|---|---|
| SYS-001 | Avvio servizio e UI | Stato healthy visibile, nessun errore critico |
| SYS-002 | Evento di test end-to-end | Evento normalizzato e persistito una sola volta |
| SYS-003 | Evento che soddisfa regola demo | Rilevamento e incidente consultabili |
| SEC-001 | UI tenta accesso DB diretto | Nessun accesso consentito/previsto |
| REC-001 | Backup e restore su copia di test | Dati e integrità verificati |
| CFG-001 | Configurazione non valida | Rifiuto controllato e fallback sicuro |
| DET-001 | Evento non corrispondente a regola | Nessun alert ingiustificato |
| DEP-001 | Installazione pulita su Windows supportato | Servizio e UI funzionanti |

## 12. Limiti dichiarati

La visibilità sugli IoT non è garantita: dipende da router, segmentazione, protocolli, permessi e telemetria disponibile. L'assenza di un alert non dimostra che un dispositivo sia sicuro. Il rilevamento è assistivo e non sostituisce aggiornamenti, firewall, backup e buone pratiche.

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

### Chiarimento dello stack

La scelta Go + Rust + Python è confermata. Non significa che ogni funzione debba essere duplicata in più linguaggi: Go orchestra e mantiene lo stato autorevole; Rust fornisce moduli specialistici di sicurezza; Python svolge attività ausiliarie di analisi. Le comunicazioni tra componenti devono usare contratti espliciti e versionati. Nessun worker può scrivere direttamente nel database.
