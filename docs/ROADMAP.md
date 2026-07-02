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

## Deployment TODO

- Připravit postupy pro:
  - baremetal / VM
  - Docker / Docker Compose
  - Kubernetes
- Připravit automatické skripty, předpisy nebo konfigurace tam, kde to dává
  smysl.
- Vyřešit konflikt aplikačních migrací a DB ownership modelu:
  - Core-managed aplikace instalované z oficiálního zdroje nebo lokálního
    balíčku může cílová HC instance validovat, sestavit, spustit a provozně
    řídit.
  - Standalone aplikace může být provozovaná mimo cílovou HC instanci, ale
    zároveň má dodržet tenant/app DB model cílové instance.
  - Je potřeba rozhodnout, kdo vlastní DB credentials, migrace, backup/restore
    hranice a lifecycle stav pro standalone aplikace.
- Zatím nepoužívat Docker Hub ani jiný public image registry; vývojové
  instalace mohou image buildovat lokálně při instalaci.
- Vývojové deploymenty držet na HTTP; HTTPS doplnit později přes proxy nebo
  ingress terminaci.
