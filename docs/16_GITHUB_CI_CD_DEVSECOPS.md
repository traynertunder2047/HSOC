# Documento 16 – GitHub, CI/CD e DevSecOps

**Progetto:** HSOC – Home Security Operations Center  
**Versione documento:** 1.1 – bozza progettuale aggiornata  
**Stato:** Specifica proposta, da approvare e implementare  
**Data:** 15 novembre 2026

---

## 1. Scopo

Questo documento definisce come organizzare il repository GitHub di HSOC e come automatizzare controlli di qualità, test, sicurezza e preparazione delle release.

Gli obiettivi sono:

- individuare errori prima che le modifiche entrino nel ramo principale;
- verificare automaticamente compilazione e test;
- controllare le dipendenze e individuare vulnerabilità note;
- ridurre il rischio di pubblicare segreti o dati personali;
- rendere riproducibili build e verifiche;
- mantenere separati sviluppo, test e distribuzione.

La CI non certifica che il prodotto sia sicuro: verifica soltanto i controlli automatizzati configurati.

## 2. Principi e vincoli

1. **Local-first:** Docker e servizi cloud non devono essere necessari per eseguire HSOC sul computer dell'utente.
2. **Costo iniziale zero:** usare le funzionalità gratuite disponibili, rispettandone limiti e condizioni. Non dipendere da servizi a pagamento.
3. **Windows first:** la prima pipeline completa deve verificare Windows. Linux e macOS saranno aggiunti quando il supporto sarà implementato.
4. **Least privilege:** workflow e token devono avere solo i permessi strettamente necessari.
5. **Fail closed per i controlli essenziali:** una verifica obbligatoria fallita blocca il merge.
6. **Riproducibilità:** versioni degli strumenti e dipendenze devono essere dichiarate e mantenute sotto controllo.
7. **Nessuna azione automatica sul sistema dell'utente:** la CI testa il software; non esegue risposte alle minacce sui dispositivi degli utenti.
8. **Nessun segreto nel repository:** token Telegram, chiavi, credenziali e dati locali non devono essere committati.

## 3. Repository e struttura

Struttura iniziale consigliata:

```text
HSOC/
├── .github/
│   ├── workflows/
│   │   ├── ci-go.yml
│   │   ├── ci-dashboard.yml
│   │   ├── ci-rust.yml
│   │   ├── ci-python.yml
│   │   ├── security.yml
│   │   └── release.yml
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
├── cmd/                     # entry point Go
├── internal/                # core e moduli Go
├── rust/                    # moduli specialistici di sicurezza
├── python/                  # worker e analisi ausiliarie
├── desktop/
│   ├── src/                 # React + TypeScript
│   └── src-tauri/           # shell Tauri
├── database/                # schema, migrations, queries
├── protocols/               # IPC, eventi, messaggi e schemi
├── rules/
├── configs/
├── installers/windows/
├── docs/
├── tests/
├── docker/                  # facoltativo: sviluppo/test
├── scripts/
├── .gitignore
├── README.md
├── SECURITY.md
├── CONTRIBUTING.md
├── LICENSE
└── CHANGELOG.md
```

Go, Rust e Python fanno parte dello stack approvato; i rispettivi moduli vengono integrati per fasi. Non è necessario creare directory o moduli vuoti prima di avviare la relativa implementazione. La struttura effettiva dovrà restare allineata all'architettura approvata nei documenti precedenti.

### 3.1 Branch

- `main`: ramo stabile; modifiche solo tramite Pull Request.
- `feature/<descrizione>`: nuove funzionalità.
- `fix/<descrizione>`: correzioni.
- `docs/<descrizione>`: documentazione.
- `security/<descrizione>`: correzioni di sicurezza, con accesso e revisione appropriati.

Per un progetto individuale, la revisione può essere una checklist personale documentata; non si deve fingere una revisione indipendente se non è avvenuta.

### 3.2 Protezione di `main`

Configurare, dove disponibile:

- obbligo di Pull Request;
- controlli CI obbligatori;
- blocco dei force-push;
- blocco della cancellazione del branch;
- risoluzione delle conversazioni prima del merge;
- almeno una revisione indipendente quando il progetto avrà più collaboratori.

Non abilitare regole che richiedono un numero di approvazioni impossibile da soddisfare per un repository gestito da una sola persona.

## 4. Strategia CI

La CI (Continuous Integration) viene eseguita sulle Pull Request e sugli aggiornamenti dei branch rilevanti. È prevista anche un'esecuzione pianificata per controlli periodici.

### 4.1 Pipeline Go – agente principale

Controlli previsti:

- formattazione con `gofmt`;
- analisi con `go vet`;
- test con `go test ./...`;
- compilazione per il target Windows supportato;
- controllo delle vulnerabilità dei moduli Go con `govulncheck`;
- verifica che `go.mod` e `go.sum` siano coerenti.

Comandi indicativi:

```bash
gofmt -w .
go vet ./...
go test ./...
govulncheck ./...
go build ./...
```

In CI, il controllo di formattazione deve verificare il repository senza modificare i file e fallire se trova file non formattati. Il comando `gofmt -w` è utile localmente, non come correzione silenziosa del commit nel workflow.

### 4.2 Pipeline Rust – moduli specialistici

Da attivare quando il codice Rust sarà introdotto:

- `cargo fmt --check`;
- `cargo clippy -- -D warnings`;
- `cargo test`;
- controllo delle dipendenze con `cargo audit`;
- build per i target effettivamente supportati.

### 4.3 Pipeline Python – moduli ausiliari

Da attivare quando Python sarà introdotto:

- lint e formattazione con uno strumento scelto e versionato, ad esempio Ruff;
- test con `pytest`;
- controllo delle dipendenze con `pip-audit`;
- build o packaging solo se previsto dal componente.

Le dipendenze devono essere fissate o bloccate con un lockfile, secondo il sistema di packaging adottato.

### 4.4 Dashboard – React e TypeScript

Da attivare quando la dashboard sarà introdotta:

- installazione riproducibile delle dipendenze tramite lockfile;
- lint;
- controllo dei tipi TypeScript;
- test automatici;
- build di produzione.

Usare `npm ci` se il progetto adotta npm e mantiene `package-lock.json`; per altri package manager usare il relativo comando deterministico.

### 4.5 Test d'integrazione

I test d'integrazione verificano i confini tra componenti, in particolare:

1. acquisizione e validazione di un evento di test;
2. normalizzazione da parte dell'agente Go;
3. persistenza nel database SQLite;
4. generazione di un evento o incidente dimostrativo;
5. visualizzazione nella dashboard, quando disponibile;
6. comportamento in caso di database non disponibile o dati non validi.

I test devono usare dati sintetici e ambienti temporanei. Non devono richiedere accesso ai dispositivi reali dell'utente.

## 5. Sicurezza della supply chain e del repository

### 5.1 Segreti

- Abilitare secret scanning e push protection, se disponibili.
- Non inserire token, password, chiavi private, file `.env` o backup nel repository.
- Usare GitHub Secrets solo per credenziali strettamente necessarie ai workflow.
- Non stampare segreti nei log.
- Revocare e sostituire immediatamente un segreto accidentalmente pubblicato: cancellarlo dal commit più recente non è sufficiente.

### 5.2 Dipendenze

- Preferire dipendenze mantenute e necessarie.
- Fissare le versioni e aggiornare in modo controllato.
- Valutare provenienza, licenza, manutenzione e vulnerabilità note.
- Usare Dependabot o strumenti equivalenti quando compatibili con il piano e il repository.
- Le pull request automatiche di aggiornamento devono superare test e controlli prima del merge.

### 5.3 Permessi dei workflow

Ogni workflow deve dichiarare esplicitamente i permessi minimi del `GITHUB_TOKEN`. Per le pipeline di test, partire da permessi di sola lettura, ad esempio `contents: read`.

Non usare `write-all`. Non eseguire codice non fidato di Pull Request con segreti disponibili. Per le Pull Request da fork, mantenere i workflow senza privilegi di scrittura e senza segreti.

### 5.4 Azioni GitHub

- Preferire azioni ufficiali o ampiamente verificate.
- Fissare le azioni a una versione stabile e, per i workflow sensibili, al commit SHA completo.
- Aggiornare periodicamente le versioni fissate.
- Non eseguire script remoti non verificati.

## 6. Scansioni di sicurezza

La pipeline di sicurezza può includere:

| Controllo | Scopo |
|---|---|
| Secret scanning | Individuare credenziali accidentalmente committate |
| Dependency scanning | Rilevare vulnerabilità note nelle dipendenze |
| Static analysis | Individuare classi di errori nel codice |
| License review | Identificare licenze incompatibili o da valutare |
| SBOM | Produrre un inventario delle componenti software |
| Container scanning | Controllare immagini Docker, se utilizzate |

La scansione non deve essere interpretata come garanzia di assenza di vulnerabilità. Ogni risultato va classificato per severità, impatto e applicabilità al progetto.

## 7. Docker: uso facoltativo

Docker è uno strumento per sviluppo e test, non un requisito di installazione o funzionamento di HSOC.

Usi consentiti e utili:

- ambienti ausiliari riproducibili;
- test d'integrazione isolati;
- esecuzione di strumenti di analisi;
- servizi temporanei di test, se in futuro necessari.

Vincoli:

- l'agente desktop HSOC non deve richiedere Docker;
- non eseguire l'agente con privilegi elevati dentro un container per aggirare i limiti del container;
- non montare directory personali o socket host senza una necessità documentata;
- non inserire credenziali nelle immagini;
- usare immagini di base affidabili, versionate e aggiornate;
- generare e controllare la SBOM delle immagini, se pubblicate.

Un `Dockerfile` o `compose.yaml` va aggiunto solo quando esiste un caso d'uso concreto. Evitare un Compose composto da servizi fittizi o non necessari.

## 8. CD e release

La Continuous Delivery (CD) prepara gli artefatti; la pubblicazione deve essere un passaggio deliberato e verificabile.

### 8.1 Condizioni per una release

Una release non può essere pubblicata come stabile se:

- i test obbligatori falliscono;
- la build non è riproducibile o non è stata verificata;
- esistono vulnerabilità bloccanti non valutate;
- mancano note di rilascio;
- non è stato verificato il percorso di installazione e disinstallazione;
- backup e ripristino non sono stati testati per le modifiche che coinvolgono i dati.

### 8.2 Artefatti

In base allo stato reale del progetto, una release potrà contenere:

- installer Windows;
- checksum degli artefatti;
- SBOM;
- note di rilascio;
- istruzioni di aggiornamento e ripristino.

La firma digitale degli installer è raccomandata prima della distribuzione pubblica. Può comportare costi o requisiti esterni: non va promessa come funzionalità gratuita garantita.

### 8.3 Versionamento

Adottare Semantic Versioning quando l'API e il prodotto saranno sufficientemente stabili:

- `MAJOR`: cambiamenti incompatibili;
- `MINOR`: funzionalità compatibili;
- `PATCH`: correzioni compatibili.

Durante lo sviluppo iniziale, usare versioni `0.x` e indicare chiaramente che il software è sperimentale.

## 9. Documentazione del repository

File minimi:

- `README.md`: scopo, stato, requisiti, avvio e struttura;
- `SECURITY.md`: come segnalare una vulnerabilità in modo responsabile;
- `CONTRIBUTING.md`: convenzioni, test e Pull Request;
- `CHANGELOG.md`: modifiche per versione;
- `LICENSE`: licenza scelta consapevolmente;
- template Issue e Pull Request.

Non scegliere una licenza per supposizione: la decisione deve essere esplicita e coerente con le dipendenze e con il modello di distribuzione.

## 10. Criteri di accettazione

La configurazione CI/CD è accettabile quando:

1. una Pull Request esegue i controlli previsti per i componenti presenti;
2. un errore in un controllo obbligatorio impedisce il merge in `main`;
3. i workflow usano permessi minimi;
4. i segreti non sono esposti nei log o nei workflow non fidati;
5. le dipendenze sono controllate;
6. i test usano dati sintetici e non modificano il sistema host;
7. Docker non è necessario per installare o avviare HSOC;
8. una release include artefatti e informazioni verificabili;
9. le scansioni e i risultati sono conservati in modo consultabile;
10. ogni controllo non ancora implementato è dichiarato come tale.

## 11. Sequenza d'implementazione

| Fase | Attività | Priorità |
|---|---|---|
| 1 | Creare repository e struttura minima | P0 |
| 2 | Aggiungere README, SECURITY, CONTRIBUTING, LICENSE da decidere e CHANGELOG | P0 |
| 3 | Configurare CI Go per formattazione, analisi, test e build | P0 |
| 4 | Proteggere `main` e rendere obbligatori i controlli CI | P0 |
| 5 | Aggiungere scansione segreti e dipendenze | P0 |
| 6 | Aggiungere test d'integrazione e test backup/restore | P0 |
| 7 | Aggiungere CI dashboard, Rust e Python quando i moduli esistono | P1 |
| 8 | Introdurre Docker solo per casi d'uso di sviluppo/test documentati | P1 |
| 9 | Preparare workflow di release e SBOM | P1 |
| 10 | Valutare firma degli installer e matrice Linux/macOS | P2 |

## 12. Decisioni ancora da confermare

| ID | Decisione | Stato |
|---|---|---|
| ADR-CI-01 | GitHub Actions come piattaforma CI | Scelta di progetto da configurare |
| ADR-CI-02 | `main` protetto con Pull Request e controlli obbligatori | Proposta |
| ADR-CI-03 | Windows come primo target CI completo | Coerente con la direzione del progetto |
| ADR-CI-04 | Docker facoltativo, non richiesto all'utente finale | Confermata |
| ADR-CI-05 | Strumenti di scansione e soglie di blocco | Da definire durante l'implementazione |
| ADR-CI-06 | Licenza del progetto | Da decidere |
| ADR-CI-07 | Strategia di firma e distribuzione degli installer | Da decidere |

## 13. Distinzione tra specificato, implementato e verificato

Questo documento descrive una configurazione proposta. La presenza di un workflow in questa specifica non significa che il workflow sia stato creato, eseguito o superato.

Per ogni componente mantenere tre stati distinti:

- **SPECIFICATO:** comportamento e controlli descritti;
- **IMPLEMENTATO:** file, codice e configurazione presenti nel repository;
- **VERIFICATO:** controlli eseguiti e risultati registrati.

Non dichiarare una release, una scansione o un test come superati senza un risultato verificabile.

---

**Fine del Documento 16 – GitHub, CI/CD e DevSecOps**


## 14. Decisioni HSOC consolidate nella revisione 1.1

- Licenza del progetto: MIT (`LICENSE` nella radice del repository).
- Repository: monorepo GitHub.
- Dashboard: Tauri + React + TypeScript.
- Linguaggi approvati: Go, Rust e Python. Go è il core e unico writer SQLite; Rust è destinato a componenti specialistici di sicurezza; Python a worker e analisi ausiliarie.
- La CI comprende workflow distinti per Go, Rust, Python e dashboard, applicati ai componenti effettivamente presenti.
- Docker è facoltativo e limitato a sviluppo/test; non è richiesto all'utente finale.
- Agenti remoti: implementazione funzionale dalla V2; nella V1 sono esclusi listener e agenti remoti.
- Windows x86-64 è il primo target; Linux/macOS verranno aggiunti dopo la stabilizzazione.

La decisione di usare tre linguaggi non implica tre processi obbligatori per ogni funzione. Ogni componente deve avere responsabilità, contratto, test e piano di packaging definiti. La documentazione non prova che i workflow siano già configurati nel repository.
