# HSOC — Home Security Operations Center

HSOC è progettato come applicazione desktop local-first per la sicurezza informatica domestica.

## Baseline approvata
- Core e servizio: Go (unico writer del database SQLite).
- Moduli specialistici: Rust.
- Worker e analisi ausiliarie: Python.
- Dashboard: Tauri + React + TypeScript.
- Database: SQLite locale.
- Docker: facoltativo, per sviluppo e test.
- V1: agente locale, Windows-first.
- V2+: agenti remoti.

## Stato del repository
Questo repository contiene lo scheletro iniziale e la documentazione di progetto.
Le cartelle con `README.md` sono segnaposto organizzativi: non indicano che le funzionalità siano già implementate.
I workflow CI saranno attivati e completati quando i rispettivi moduli e manifest di dipendenze saranno presenti.

## Documentazione
Vedere [`docs/00_INDICE_PACCHETTO.md`](docs/00_INDICE_PACCHETTO.md).

## Licenza
MIT — vedere [`LICENSE`](LICENSE).
