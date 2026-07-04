# Hekatoncheiros Roadmap

## Phase 1 – Local Installable Apps

- DB-per-tenant model (v MVP logicky v jedné fyzické DB)
- schema-per-app (`app_<app_id>`)
- migration via core (`/.well-known/hc/migrations*`)
- hash-verified migrations (SHA-256, MVP)
- licensing enforcement
- signed manifests
- installation lifecycle
- local application catalog entries
- catalog feed data model
- catalog install modes: external, stage-only, and planned Core-managed compose
- manual catalog feed source sync

## Phase 2 – Core2Core Integration

- remote apps
- scheduled application catalog feed sync
- namespace trust / protected namespace policy
- OAuth trust
- no shared DB
- signed cross-core tokens
- signed migration bundles

> Core2Core není v současném scope implementováno.

## Deployment TODO

- Udržet Docker Compose jako aktuální ověřitelnou cestu pro lokální self-host.
- Baremetal / VM a Kubernetes zatím držet jen jako stručné skici.
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
- Navrhnout a implementovat Core runtime manager pro aplikační compose balíčky:
  bezpečné umístění compose souborů, allowlist publish/volume/network pravidel,
  auditované schválení adminem a lifecycle start/stop/update.
- Vývojové deploymenty držet na HTTP; HTTPS doplnit později přes proxy nebo
  ingress terminaci.
- Zrevidovat dlouhodobou hranici `hc-author-registry`: zatím zůstává oddělené,
  ale může dávat větší smysl jako volitelný authority mode v Core.
