# HSOC — Release & Maintenance Plan

**Versione:** 1.1-draft

## 1. Versionamento

Adottare Semantic Versioning:
- MAJOR: incompatibilità API/schema/policy.
- MINOR: funzionalità compatibili.
- PATCH: correzioni compatibili.

Ogni release deve indicare versione servizio, UI, protocollo IPC, schema DB e set regole.

## 2. Canali

- Development: build interne e test.
- Preview: distribuzione limitata per verifica.
- Stable: solo dopo gate completi.

Non distribuire build non firmate come release stable.

## 3. Pipeline release

1. Freeze feature.
2. Verifica issue e rischi aperti.
3. Test completi.
4. Scan dipendenze e segreti.
5. SBOM.
6. Build e firma.
7. Test installazione/aggiornamento.
8. Test backup/restore.
9. Note di rilascio e known issues.
10. Pubblicazione e monitoraggio.

## 4. Manutenzione vulnerabilità

- Controllo mensile di OS e dipendenze utilizzate.
- Controllo fuori ciclo per advisory critici o exploit ampiamente rilevanti.
- Registrare advisory, componente, versione, impatto, mitigazione e scadenza.
- Aggiornare in ambiente di test prima del rilascio.
- Non applicare auto-fix al codice di produzione senza revisione umana.
- Auto-fix ammesso solo come patch proposta, con test e review.

## 5. Dipendenze e SBOM

- Inventario dipendenze dirette/transitive.
- Lockfile versionati.
- Rimozione dipendenze inutilizzate.
- Licenze verificate.
- SBOM per ogni release stabile.
- Scanner in CI e report conservati.

## 6. Supporto e bug triage

Classificare:
- Critical: perdita dati, esecuzione non autorizzata, compromissione o install/update insicuro.
- High: feature core indisponibile o rischio significativo.
- Medium: degradazione con workaround.
- Low: difetto cosmetico o minore.

Ogni bug deve indicare versione, ambiente, passaggi, expected/actual, log redatto, requisito e test collegati.

## 7. Compatibilità

Mantenere una matrice testata con:
- versione Windows;
- architettura;
- versione Go runtime/build;
- versione UI/Tauri;
- versione schema DB;
- versione protocollo;
- modalità upgrade.

Non dichiarare supporto solo perché il software sembra avviarsi.

## 8. Rollback e migrazioni

- Backup prima di migrazioni.
- Verificare restore su release candidate.
- Rollback binario e strategia DB documentati.
- Se rollback schema non è sicuro, richiedere restore del backup.
- Non cancellare automaticamente il backup pre-update.

## 9. Test manutenzione

- MNT-01: dependency scan mensile.
- MNT-02: advisory critico fuori ciclo.
- MNT-03: SBOM generata e archiviata.
- MNT-04: firma artefatti verificata.
- MNT-05: upgrade da release precedente.
- MNT-06: rollback/recovery.
- MNT-07: migrazione non distruttiva.
- MNT-08: changelog e known issues.
- MNT-09: ADR aggiornate.
- MNT-10: patch auto-fix sottoposta a review e test.

## 10. Criteri di fine supporto

Comunicare con anticipo la fine supporto di una versione. Non forzare aggiornamenti distruttivi. Fornire istruzioni backup, upgrade e migrazione.

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

### Manutenzione dei componenti e licenze

La scansione periodica deve coprire separatamente:
- moduli Go (`go.mod`/`go.sum`);
- crate Rust (`Cargo.toml` e lockfile);
- dipendenze Python e relativo lockfile;
- dipendenze frontend Node/Tauri e lockfile;
- immagini Docker soltanto se utilizzate nei workflow o pubblicate.

Per ogni release generare un inventario SBOM dei componenti distribuiti e verificare la compatibilità delle licenze. La licenza MIT di HSOC non cambia né sostituisce le licenze dei componenti di terze parti. Conservare avvisi e attribuzioni in `THIRD_PARTY_NOTICES.md` quando richiesto.
