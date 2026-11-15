# HSOC — API & IPC Specification

**Versione:** 1.1-draft | **Stato:** contratti da consolidare

## 1. Obiettivo

Definire la comunicazione tra dashboard desktop e servizio Go locale, senza accesso diretto al database. L'IPC è un confine di sicurezza: input della UI sempre non fidato.

## 2. Architettura

- V1 Windows: valutare Named Pipes locali.
- ACL restrittive: solo account/sessione autorizzati.
- Il servizio verifica identità/permessi del client, non si fida del solo nome del pipe.
- Nessun listener TCP esposto per default.
- UI può connettersi/disconnettersi senza fermare il servizio.
- Il servizio può funzionare senza UI.
- Protocollo con versione esplicita e compatibilità controllata.

## 3. Famiglie di operazioni

| Famiglia | Operazioni |
|---|---|
| Health | `GetHealth`, `GetVersion`, `GetCapabilities` |
| Events | `ListEvents`, `GetEvent` |
| Detection | `ListDetections`, `GetDetection` |
| Incidents | `ListIncidents`, `GetIncident`, `UpdateIncidentStatus` |
| Inventory | `ListInventory`, `GetInventoryItem` |
| Vulnerability | `ListFindings`, `GetScanStatus`, `RequestScan` |
| Configuration | `GetConfig`, `ValidateConfig`, `ApplyConfig` |
| Rules | `ListRules`, `SetRuleEnabled` |
| Backup | `CreateBackup`, `ListBackups`, `VerifyBackup`, `RestoreBackup` |
| Diagnostics | `GetDiagnostics`, `ExportDiagnostics` |
| Simulation | `RunSimulation`, `GetSimulationResult` |
| Notifications | `GetNotificationStatus`, `TestNotification` |

Le operazioni distruttive o sensibili richiedono autorizzazione, audit e, dove previsto, approvazione esplicita.

## 4. Contratto messaggio

Formato logico consigliato:
```json
{
  "protocol_version": "1",
  "request_id": "unique-id",
  "operation": "GetHealth",
  "payload": {},
  "client_metadata": {
    "ui_version": "x.y.z"
  }
}
```

Risposta:
```json
{
  "protocol_version": "1",
  "request_id": "same-id",
  "ok": true,
  "data": {},
  "error": null
}
```

I nomi e i tipi sono illustrativi: lo schema formale deve essere versionato e validato. Non inserire segreti nei metadata.

## 5. Error model

Codici logici:
- `INVALID_ARGUMENT`
- `UNAUTHENTICATED`
- `PERMISSION_DENIED`
- `NOT_FOUND`
- `CONFLICT`
- `FAILED_PRECONDITION`
- `RESOURCE_EXHAUSTED`
- `UNAVAILABLE`
- `INTERNAL`
- `UNSUPPORTED_VERSION`

Gli errori non devono esporre stack trace, path sensibili, token o dettagli interni alla UI. I log tecnici possono contenere correlation ID e contesto redatto.

## 6. Regole IPC

1. Validare dimensione massima del messaggio.
2. Validare schema, enum, range e encoding.
3. Applicare timeout e cancellazione.
4. Limitare richieste concorrenti e rate.
5. Rifiutare versioni non supportate.
6. Non eseguire shell command da input IPC.
7. Autorizzare ogni operazione sensibile sul lato Go.
8. Auditare cambi di configurazione, policy, restore e azioni.
9. Nessun segreto in chiaro nei log.
10. Chiudere il canale in modo pulito e recuperare da UI riavviata.

## 7. Autorizzazione

- Nessun login HSOC dedicato nella V1.
- Usare l'identità Windows e ACL del canale locale.
- La UI non è automaticamente trusted: il servizio valida ogni richiesta.
- Per operazioni che richiedono elevazione, preferire helper/operazione circoscritta e documentata; non rendere tutto il servizio amministratore.
- Confermare esplicitamente restore, modifiche policy e azioni con impatto.

## 8. Eventi push

Possibili notifiche dal servizio:
- `health.changed`
- `event.received`
- `detection.created`
- `incident.updated`
- `scan.completed`
- `backup.completed`
- `configuration.changed`

Gli eventi push sono informativi; la UI deve poter recuperare lo stato tramite query se perde una notifica.

## 9. Versionamento

- `protocol_version` obbligatoria.
- Aggiunte compatibili preferite; rimozioni/semantica diversa richiedono versione nuova.
- UI e servizio negoziano capabilities.
- Test di compatibilità tra UI N e servizio N-1 secondo matrice release.
- Contratti e tipi generati o verificati da schema condiviso.

## 10. Test IPC

| ID | Test | Risultato atteso |
|---|---|---|
| IPC-001 | UI autorizzata si connette | Connessione stabilita |
| IPC-002 | Client non autorizzato | Connessione/richiesta negata |
| IPC-003 | Payload malformato | Rifiuto controllato |
| IPC-004 | Payload oltre limite | Rifiuto senza consumo eccessivo |
| IPC-005 | Timeout servizio | UI mostra stato non disponibile |
| IPC-006 | Versione protocollo sconosciuta | `UNSUPPORTED_VERSION` |
| IPC-007 | Request ID duplicato | Comportamento deterministico documentato |
| IPC-008 | Richiesta non autorizzata di restore | Negata e auditata |
| IPC-009 | UI chiusa durante push | Servizio continua, UI recupera stato |
| IPC-010 | Input contenente comandi | Trattato come dato, mai eseguito |
| IPC-011 | Flood di richieste | Rate/concurrency limit applicati |
| IPC-012 | Segreti in errore | Nessuna divulgazione nei messaggi/log |

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

### Compatibilità con lo stack approvato

L'IPC desktop V1 collega la dashboard Tauri/React/TypeScript al servizio Go. I moduli Rust specialistici e i worker Python comunicano con Go tramite interfacce interne ristrette e versionate; non devono essere trattati come client privilegiati. Ogni messaggio proveniente da un componente separato è validato come input non fidato.

Gli schemi in `protocols/` devono distinguere:
- IPC locale dashboard-servizio V1;
- contratti interni Go-Rust e Go-Python;
- schemi di eventi e dati;
- futuri contratti agente remoto V2, disattivati e non esposti nella V1.
