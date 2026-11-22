# Contributing

1. Open or link an issue/requirement.
2. Create a descriptive branch.
3. Keep the Pull Request limited to a single purpose.
4. Add/update tests and documentation.
5. Update the traceability matrix when a requirement changes.
6. Do not include secrets, personal data, or local databases.
7. Wait for CI checks and review before merging.

## Stack
- Go: core and SQLite access.
- Rust: specialist security modules.
- Python: worker/auxiliary analysis.
- Tauri + React + TypeScript: dashboard.

Rust/Python modules do not access SQLite directly. Docker is optional and intended for development/testing.

## License
Contributions to the project are proposed under the repository MIT license, subject to the applicable conditions and third-party notices.
