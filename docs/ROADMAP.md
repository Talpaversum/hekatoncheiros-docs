# Hekatoncheiros Roadmap

## Phase 1 – Local Installable Apps

- DB-per-tenant model (v MVP logicky v jedné fyzické DB)
- schema-per-app (`app_<app_id>`)
- migration via core (`/.well-known/hc/migrations*`)
- hash-verified migrations (SHA-256, MVP)
- licensing enforcement for tenant runtime navigation/API access
- signed manifests
- installation lifecycle
- local application catalog entries
- catalog feed data model
- catalog install modes: external, stage-only, and planned Core-managed compose
- manual catalog feed source sync
- public instance app feed export
- admin publish/unpublish controls for installed apps
- Core-owned UI plugin artifact storage and manual artifact refresh
- Core Console management UI for catalog, installed apps, tenant/platform settings,
  RBAC/user/tenant management, licensing activation, toast feedback, and Help
  dropdown sections
- manifest-provided procedural Help entries (`integration.ui.help_entries`)

## Phase 2 – Core2Core Integration

- remote apps
- scheduled application catalog feed sync
- "suggest app for feed" workflow with pending publication requests
- admin-issued publish tokens for pre-approved namespaces/apps/CI pipelines
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
- Rozšířit lifecycle pro aplikační artefakty:
  - ruční admin akce `Refresh artifact` v managementu nainstalovaných aplikací
    je hotová,
  - ruční admin akce `Check update` pro porovnání aktuálního manifest hashe s
    uloženou instalací je hotová,
  - pasivní katalogový signál `catalog_update` v seznamu nainstalovaných aplikací
    je hotový,
  - notifikace z aplikace nebo feedu, že manifest/UI artefakt má novou verzi,
  - volitelný automatický refresh pro trusted/official zdroje,
  - audit a policy guard pro automatické změny runtime UI.
- Vývojové deploymenty držet na HTTP; HTTPS doplnit později přes proxy nebo
  ingress terminaci.
- Zrevidovat dlouhodobou hranici `hc-author-registry`: zatím zůstává oddělené,
  ale může dávat větší smysl jako volitelný authority mode v Core.
- Zapojit `hc-author-registry` do licenčního author onboardingu:
  - lokální dev flow nesmí dlouhodobě nahrazovat registry falešnou autoritou,
  - registry má vydávat `author_id`, registrovat autorovy veřejné klíče,
    vydávat `author_cert_jws` a publikovat root JWKS/revocation snapshoty,
  - `hc-app-licensing` má používat certifikát vydaný registry místo ručně
    generovaného dev certifikátu.
- Rozhodnout produktovou hranici `hc-app-licensing`:
  - dnes je to backend issuer bez UI a bez aplikačního manifestu,
  - navrhnout, zda má zůstat samostatnou autor/vendor službou, nebo dostat
    instalovatelný Hekatoncheiros app manifest a Core Console UI,
  - pokud bude instalovatelná, doplnit manifest, compose balíček, UI plugin a
    administraci zákazníků, grantů, vydaných licencí a revoke workflow.
