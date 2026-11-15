# HSOC — User & Administrator Guide

**Versione:** 1.1-draft | **Destinatari:** utenti locali e manutentori

## 1. Introduzione

HSOC raccoglie e organizza segnali di sicurezza disponibili sul computer. La dashboard permette di consultare eventi, rilevamenti, incidenti, inventario e stato del servizio. Il prodotto è assistivo: non garantisce che ogni minaccia venga individuata.

## 2. Avvio e stato

La UI mostra:
- servizio disponibile/non disponibile;
- stato dei collector;
- ultimo evento acquisito;
- stato DB;
- ultimo backup;
- eventuali controlli vulnerabilità in corso;
- versione HSOC e versione protocollo.

Se il servizio è indisponibile, la UI deve mostrare un messaggio comprensibile e un ID diagnostico, non stack trace o segreti.

## 3. Dashboard

Sezioni consigliate:
1. Panoramica.
2. Eventi.
3. Rilevamenti.
4. Incidenti.
5. Inventario software.
6. Vulnerabilità.
7. Backup e ripristino.
8. Configurazione.
9. Notifiche.
10. Stato e diagnostica.

Ogni rilevamento deve mostrare fonte, data, regola/versione, severità, confidenza, motivazione e azioni suggerite.

## 4. Incidenti

Stati suggeriti:
- `OPEN`
- `ACKNOWLEDGED`
- `INVESTIGATING`
- `RESOLVED`
- `FALSE_POSITIVE`

La chiusura come falso positivo deve conservare la traccia e, se possibile, il motivo. Non cancellare silenziosamente eventi.

## 5. Configurazione

- Le impostazioni devono spiegare l'effetto di ogni opzione.
- Opzioni rischiose off per default.
- Le policy di sicurezza non disattivabili non devono apparire come toggle ordinari.
- Config invalida non deve sostituire l'ultima configurazione valida.
- Le modifiche importanti sono auditate.

## 6. Telegram e notifiche

Telegram è opzionale e disattivato inizialmente. L'attivazione deve:
- essere esplicita;
- spiegare quali dati vengono inviati e a quale servizio;
- memorizzare token nel Credential Store del sistema operativo;
- non inviare payload sensibili non necessari;
- permettere test e revoca;
- non accettare comandi remoti come risposta a notifiche.

## 7. IoT

HSOC può mostrare soltanto informazioni ottenibili in modo passivo o tramite fonti supportate. Alcuni router non espongono dati sufficienti. Dispositivo non rilevato non significa dispositivo sicuro; dispositivo rilevato non significa dispositivo compromesso.

## 8. Backup e restore

- Creare backup prima di aggiornamenti importanti.
- Verificare checksum e stato backup.
- Eseguire periodicamente un restore di prova su ambiente separato.
- Prima del restore, mostrare quale DB verrà sostituito e quali dati potrebbero andare persi.
- Non sovrascrivere il DB attivo senza conferma.

## 9. Diagnostica

Export diagnostico:
- deve essere volontario;
- deve mostrare anteprima dei contenuti;
- deve redigere token, credenziali e dati non necessari;
- deve includere versioni, health, errori e informazioni tecniche minime;
- non deve inviare automaticamente il file a terzi.

## 10. Risoluzione problemi

| Sintomo | Controllo |
|---|---|
| UI non si connette | Verificare servizio e ACL IPC |
| Servizio non parte | Consultare log redatto e health |
| Nessun evento | Verificare stato collector e limiti OS |
| Troppi alert | Esaminare regola, versione, confidence e fixture |
| Backup fallito | Spazio, permessi, WAL e checksum |
| Restore non disponibile | Verificare versione DB e integrità backup |
| Telegram non invia | Stato opt-in, token e connettività |

## 11. Privacy

I dati sono locali per default. Prima di abilitare integrazioni esterne, spiegare i dati condivisi. L'utente deve poter disattivare Telegram e rimuovere le credenziali. Retention ed export devono essere trasparenti.

## 12. Test usabilità e guide

- UI-01: panoramica mostra health corretto.
- UI-02: evento e incidente sono distinguibili.
- UI-03: severity e confidence hanno definizioni accessibili.
- UI-04: errore IPC non causa schermata bloccata.
- UI-05: impostazioni opzionali off al primo avvio.
- UI-06: restore mostra conferma e conseguenze.
- UI-07: export mostra anteprima/redazione.
- UI-08: limiti IoT sono espliciti.
- UI-09: guida installazione coincide con installer reale.
- UI-10: documentazione aggiornata per release.

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

### Nota per l'utente finale

L'utente non deve installare Docker né toolchain di sviluppo (Go, Rust, Python, Node.js). L'installer HSOC distribuirà i componenti necessari. La dashboard è un'applicazione desktop Tauri con interfaccia React/TypeScript; il servizio Go può continuare a funzionare anche quando la dashboard è chiusa. I moduli interni Rust/Python non richiedono configurazioni separate da parte dell'utente.
