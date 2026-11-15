# Contributing

1. Aprire o collegare una issue/requisito.
2. Creare un branch descrittivo.
3. Limitare la Pull Request a uno scopo.
4. Aggiungere/aggiornare test e documentazione.
5. Aggiornare la matrice di tracciabilità quando cambia un requisito.
6. Non includere segreti, dati personali o database locali.
7. Attendere i controlli CI e la review prima del merge.

## Stack
- Go: core e accesso SQLite.
- Rust: moduli specialistici di sicurezza.
- Python: worker/analisi ausiliarie.
- Tauri + React + TypeScript: dashboard.

I moduli Rust/Python non accedono direttamente a SQLite. Docker è facoltativo e destinato a sviluppo/test.

## Licenza
I contributi al progetto sono proposti sotto la licenza MIT del repository, fatte salve le condizioni applicabili e gli avvisi di terze parti.
