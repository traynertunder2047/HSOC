# HSOC — Indice del pacchetto documentale

**Versione documentale:** 1.1-draft  
**Data:** 2026-11-15  
**Stato:** Bozza tecnica consolidata — non equivale a software implementato o verificato.

## Contenuto

| File | Contenuto |
|---|---|
| `01_MASTER_TECHNICAL_BLUEPRINT.md` | Visione, requisiti, architettura e confini del sistema |
| `02_DATABASE_SPECIFICATION.md` | Modello dati SQLite, integrità, migrazioni, retention e backup |
| `03_API_IPC_SPECIFICATION.md` | Contratti interni, IPC, eventi, errori e versionamento |
| `04_SECURITY_THREAT_MODEL.md` | Asset, minacce, confini di fiducia, mitigazioni e test di sicurezza |
| `05_DEPLOYMENT_INSTALLATION.md` | Installazione Windows, servizio, dashboard, aggiornamenti e disinstallazione |
| `06_IMPLEMENTATION_BLUEPRINT.md` | Moduli, struttura repository, stack e sequenza di sviluppo |
| `07_TESTING_QA.md` | Strategia QA, test case, metriche e criteri di rilascio |
| `08_USER_ADMIN_GUIDE.md` | Uso, configurazione, privacy, diagnosi e ripristino |
| `09_RELEASE_MAINTENANCE_PLAN.md` | Versioni, manutenzione, vulnerabilità, SBOM e release |
| `10_DETECTION_RESPONSE_SPECIFICATION.md` | Raccolta, rilevamento, incidenti, risposta e falsi positivi |
| `11_CONFIGURATION_POLICY_SPECIFICATION.md` | Configurazione, policy, default sicuri e gestione modifiche |
| `12_IMPLEMENTATION_READINESS.md` | Consolidamento, Definition of Ready/Done, vertical slice e gate |
| `13_DEVELOPMENT_RULES.md` | Regole obbligatorie durante progettazione e sviluppo |
| `14_TRACEABILITY_MATRIX.md` | Matrice requisito-documento-modulo-test-versione |
| `15_HSOC_MASTER_SUMMARY.md` | Riepilogo completo e roadmap integrata |
| `16_GITHUB_CI_CD_DEVSECOPS.md` | Repository, GitHub Actions, controlli sicurezza, Docker facoltativo e release |

## Convenzioni di stato

- **SPECIFICATO:** requisito o comportamento descritto nei documenti.
- **DA CONFERMARE:** scelta tecnica non ancora approvata definitivamente.
- **DA IMPLEMENTARE:** non si presume che il codice esista.
- **DA TESTARE:** il test è definito ma non è stata fornita evidenza di esecuzione.
- **SUPERATO:** decisione o requisito sostituito da una revisione successiva.

## Convenzioni di priorità e versione

- **P0:** necessario per MVP funzionante e sicuro.
- **P1:** importante per la prima release stabile.
- **P2:** miglioramento facoltativo della V1.
- **P3:** funzionalità futura, normalmente V2+.
- **V1-MVP:** prima slice verticale dimostrabile.
- **V1:** prima release stabile.
- **V2+:** evoluzione successiva.

## Regola fondamentale

Questo pacchetto è una baseline progettuale. Nessuna voce marcata come specificata dimostra che sia stata implementata; nessun test definito dimostra che sia stato superato. Ogni implementazione e ogni esito devono essere accompagnati da commit/versione, log o report riproducibile.

## Decisioni consolidate nella revisione 1.1

- Licenza MIT.
- Monorepo GitHub.
- Dashboard Tauri + React + TypeScript.
- Stack Go + Rust + Python: Go core/SQLite, Rust sicurezza specialistica, Python worker e analisi.
- Docker facoltativo e non richiesto all'utente finale.
- Agenti remoti implementati dalla V2; nessuna funzione remota attiva in V1.
- Documento 16 incluso nel pacchetto.
- File di supporto repository inclusi nel pacchetto ZIP: `LICENSE`, `README.md`, `.gitignore`, `SECURITY.md`, `CONTRIBUTING.md` e altri.

La revisione 1.1 aggiorna la baseline documentale. Non certifica codice o test implementati.


## File di supporto repository inclusi

- `LICENSE`: testo MIT.
- `README.md`: descrizione del progetto e dello stack.
- `.gitignore`, `.editorconfig`, `.gitattributes`: convenzioni e file esclusi.
- `SECURITY.md`: procedura iniziale per segnalazioni di sicurezza.
- `CONTRIBUTING.md`: regole per contributi e Pull Request.
- `CODE_OF_CONDUCT.md`: regole di collaborazione.
- `CHANGELOG.md`: cronologia delle revisioni documentali.
- `THIRD_PARTY_NOTICES.md`: inventario/attribuzioni di terze parti da completare.
- `NOTICE`, `AUTHORS.md`, `SUPPORT.md`: avvisi e governance iniziale.
- `MANIFEST.json`: elenco file, dimensioni e SHA-256.

I file di governance sono bozze iniziali: prima della pubblicazione del repository andranno completati i contatti e i canali effettivi di supporto e disclosure.
