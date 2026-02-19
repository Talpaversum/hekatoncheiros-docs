# Hekatoncheiros Roadmap

## Phase 1 – Local Installable Apps

- DB-per-tenant model (v MVP logicky v jedné fyzické DB)
- schema-per-app (`app_<app_id>`)
- migration via core (`/.well-known/hc/migrations*`)
- hash-verified migrations (SHA-256, MVP)
- licensing enforcement
- signed manifests
- installation lifecycle

## Phase 2 – Core2Core Integration

- remote apps
- OAuth trust
- no shared DB
- signed cross-core tokens
- signed migration bundles

> Core2Core není v současném scope implementováno.
