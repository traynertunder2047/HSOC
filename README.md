# HSOC — Home Security Operations Center

HSOC is designed as a local-first desktop application for home cybersecurity.

## Approved baseline
- Core and service: Go (the only writer of the SQLite database).
- Specialist modules: Rust.
- Workers and auxiliary analysis: Python.
- Dashboard: Tauri + React + TypeScript.
- Database: local SQLite.
- Docker: optional, for development and testing.
- V1: local agent, Windows-first.
- V2+: remote agents.

## Repository status
This repository contains the initial skeleton and the project documentation.
Folders with a `README.md` are organizational placeholders: they do not indicate that the features are already implemented.
CI workflows will be enabled and completed once the respective modules and dependency manifests are present.

## Documentation
See [`docs/00_PACKET_INDEX.md`](docs/00_PACKET_INDEX.md).

## License
MIT — see [`LICENSE`](LICENSE).
