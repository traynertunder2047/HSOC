# HSOC — Security Threat Model

**Versione:** 1.1-draft | **Metodo:** asset → minaccia → controllo → test

## 1. Scopo e confini

Modellare le minacce al servizio locale, alla UI, al database, alle configurazioni, ai backup, alle notifiche e alle fonti di eventi. Il modello va aggiornato quando cambiano architettura, permessi, IPC o funzionalità.

## 2. Asset

- Integrità e disponibilità del DB.
- Configurazioni e policy.
- Credenziali Telegram e altri token opzionali.
- Log, eventi, incidenti e inventario.
- Integrità degli aggiornamenti e dei binari.
- Canale IPC.
- Backup e chiavi di cifratura.
- Identità del servizio e autorizzazioni.

## 3. Attori e superfici

- Utente locale autorizzato.
- Processo locale non autorizzato o compromesso.
- Input/eventi OS malformati.
- Dipendenze vulnerabili.
- Pacchetto di aggiornamento manomesso.
- Dispositivo IoT non affidabile.
- Canale di notifica esterno.

Superfici: installer, servizio, IPC, parser eventi, configurazione, DB, backup, updater, UI, integrazioni.

## 4. Minacce e mitigazioni

| ID | Minaccia | Mitigazione primaria |
|---|---|---|
| TH-01 | Processo locale non autorizzato usa IPC | ACL, verifica identità, autorizzazione per operazione |
| TH-02 | UI invia input malformato | Schema validation, limiti, fuzz/unit test |
| TH-03 | Evento costruito male causa crash | Parser robusto, limiti, isolamento errori |
| TH-04 | Privilegi eccessivi del servizio | Least privilege, capability esplicite |
| TH-05 | DB corrotto o modificato | ACL file, integrità, backup verificati |
| TH-06 | Segreti nei log/config/backup | OS Credential Store, redazione, cifratura |
| TH-07 | Update manomesso | Firma e verifica pacchetti |
| TH-08 | Dipendenza vulnerabile | Inventario, advisory scan, aggiornamento controllato |
| TH-09 | Falso positivo induce risposta errata | Confidence, motivazione, approvazione umana |
| TH-10 | Notifica falsificata usata come comando | Notifiche solo informative, nessun comando remoto |
| TH-11 | IoT non osservabile | Dichiarare limiti, non inferire sicurezza dall'assenza dati |
| TH-12 | Backup non ripristinabile | Checksum, test restore periodici |
| TH-13 | Export diagnostico divulga dati | Redazione, preview, minimizzazione |
| TH-14 | Modulo opzionale compromesso | Isolamento, interfaccia stretta, niente accesso DB |
| TH-15 | Config alterata indebolisce policy | Validazione, audit, vincoli di sicurezza non disattivabili |

## 5. Principi di controllo

- Secure defaults e fail-safe.
- Minimo privilegio e separazione UI/servizio.
- Nessun comando arbitrario.
- Input non fidati validati.
- Segreti conservati nel sistema operativo.
- Aggiornamenti verificati.
- Audit per operazioni sensibili.
- Risposte automatiche disabilitate inizialmente.
- Backup testati.
- Nessuna porta remota aperta per default.

## 6. Gestione degli incidenti di sicurezza del software

1. Rilevare e classificare il problema.
2. Conservare log e versione interessata, senza raccogliere dati non necessari.
3. Limitare la funzione vulnerabile con una policy sicura.
4. Preparare patch e test.
5. Revisione indipendente.
6. Pubblicare advisory e aggiornamento firmato quando necessario.
7. Verificare la correzione e aggiornare il threat model.

## 7. Test di sicurezza

| ID | Test | Risultato atteso |
|---|---|---|
| SEC-01 | Client IPC senza permesso | Accesso negato |
| SEC-02 | Input con campi inattesi | Rifiuto o ignoranza controllata secondo schema |
| SEC-03 | Payload enorme | Rifiuto entro limiti |
| SEC-04 | Tentativo di shell injection nei campi | Nessun comando eseguito |
| SEC-05 | Ricerca token nei log | Nessun segreto trovato |
| SEC-06 | Verifica permessi directory dati | Solo identità autorizzate |
| SEC-07 | Config che disabilita controllo di sicurezza obbligatorio | Rifiutata |
| SEC-08 | Pacchetto update non firmato/alterato | Installazione bloccata |
| SEC-09 | Export diagnostico | Segreti e dati non necessari redatti |
| SEC-10 | Worker opzionale non disponibile | Core resta stabile e segnala degrado |
| SEC-11 | Errore DB durante operazione | Nessuna scrittura parziale non consistente |
| SEC-12 | Restore non autorizzato | Negato e auditato |
| SEC-13 | Notifica con contenuto ostile | Visualizzata come testo, mai eseguita |
| SEC-14 | Servizio avviato senza privilegi elevati non necessari | Funzioni base operative |
| SEC-15 | Dipendenze note vulnerabili in CI | Pipeline segnala/blocca secondo severità |
| SEC-16 | Test di regressione su permessi | Nessun ampliamento non documentato |

## 8. Rischi residui

- Un endpoint già compromesso può falsificare o sopprimere eventi.
- Router consumer possono non esporre telemetria sufficiente.
- Un utente con privilegi amministrativi può alterare il sistema locale.
- Nessun prodotto può garantire detection completa.
- Il modello non sostituisce un audit professionale prima di una distribuzione pubblica.

## 9. Gate

Prima di ogni release: threat model aggiornato, test SEC eseguiti, vulnerabilità note valutate, firma verificata, backup/restore testato, nessun rischio critico aperto senza accettazione formale.

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

### Rischi aggiuntivi introdotti dallo stack multi-linguaggio

- **Confini Go/Rust/Python:** validare schema, dimensioni, encoding e versioni a ogni confine; gestire timeout, crash e risultati incompleti.
- **FFI Rust:** mantenere interfacce piccole, documentare ownership e lifetime, gestire panic boundary e non trasferire privilegi impliciti.
- **Worker Python:** esecuzione isolata e limitata; nessuna esecuzione di codice generato o comandi arbitrari; dipendenze bloccate e verificate.
- **Dashboard Tauri:** esporre solo capability necessarie, ridurre i comandi Tauri disponibili alla UI e non considerare il frontend un confine di autorizzazione.
- **Docker:** non necessario nel prodotto; i container di test non devono ricevere segreti o montaggi host non indispensabili.
- **V2 agenti remoti:** richiederanno threat model dedicato per identità, enrollment, revoca, autenticazione reciproca, replay protection e aggiornamenti.
