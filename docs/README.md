# HSOC – Home Security Operations Center

HSOC è progettato come applicazione desktop local-first per il monitoraggio e la gestione della sicurezza del computer domestico, con visibilità IoT limitata alle informazioni disponibili senza hardware aggiuntivo.

## Stack approvato
- Go: servizio e core, unica autorità di scrittura su SQLite.
- Rust: componenti specialistici di sicurezza.
- Python: worker e analisi ausiliarie.
- Tauri + React + TypeScript: dashboard desktop.
- SQLite: archivio locale.
- Docker: facoltativo per sviluppo e test.

## Ambito versioni
- V1: agente locale singolo, Windows come primo target.
- V2+: agenti remoti e coordinamento distribuito.

## Documentazione
Consultare `00_INDICE_PACCHETTO.md` e i documenti numerati.

## Stato
Specifiche progettuali. Non dichiarare funzionalità implementate o test superati senza evidenze verificabili.

## Licenza
MIT; vedere `LICENSE`. Le dipendenze di terze parti mantengono le rispettive licenze.
