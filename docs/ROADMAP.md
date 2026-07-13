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
- catalog install modes: external, stage-only, and opt-in Core-managed compose
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
- Core runtime manager pro aplikační compose balíčky:
  - hotovo v Core:
    - validovaný deployment plán (`deployment.package_url`,
      `deployment.package_sha256`, `compose_file`, `service_name`,
      `internal_base_url`),
    - stažení runtime balíčku přes `stage_only + stage_package`,
    - ověření SHA-256 a bezpečné rozbalení `tar.gz` balíčku,
    - odmítnutí unsafe cest, symlinků, hardlinků a jiných než běžných
      souborů/adresářů,
    - základní compose policy guard: žádné published ports, host mounty,
      `privileged`, `cap_add`, `network_mode`, `container_name`, host `pid/ipc`,
    - opt-in Docker Compose adapter přes `APP_RUNTIME_DOCKER_ENABLED=true`,
    - první reálný runtime package pro `hc-app-inventory` včetně
      `docker-compose.app.yml`, zdrojů pro lokální build a SHA-256,
    - lokální vývojový katalog a manifest dostupný ještě před spuštěním
      aplikace,
    - katalogový `deployment` záznam pro Inventory:
      `type=compose`, `package_url`, `package_sha256`, `compose_file`,
      `service_name=inventory`, `internal_base_url=http://inventory:4010`,
    - ověřený `stage_only + stage_package=true` i plný `mode=compose`,
    - sdílená síť, předání `INSTALLER_TOKEN_SECRET`, DB credentials a
      dostupnost manifestu z Core runtime,
    - čekání na aplikační healthcheck před validací manifestu a dokončením
      instalace,
    - perzistentní ownership Core-managed runtime a runtime-aware uninstall,
      který před smazáním instalace odstraní vlastněné Compose kontejnery.
  - další krok: navrhnout auditované admin schválení a lifecycle
    `stop/update`.
- Rozšířit lifecycle pro aplikační artefakty:
  - ruční admin akce `Refresh artifact` v managementu nainstalovaných aplikací
    je hotová,
  - ruční admin akce `Check update` pro porovnání aktuálního manifest hashe s
    uloženou instalací je hotová,
  - pasivní katalogový signál `catalog_update` v seznamu nainstalovaných aplikací
    je hotový,
  - souhrnný admin panel pro katalogové update/stale signály v Apps managementu
    je hotový,
  - ruční admin akce `Refresh catalog` pro obnovení katalogového záznamu z
    nainstalované aplikace je hotová,
  - uložený `update_signal` pro hlášení nové verze manifestu/UI artefaktu z
    aplikace nebo feed vrstvy je hotový v admin-gated MVP podobě,
  - app-auth webhook `POST /api/v1/apps/installed/update-signal` pro
    `update_signal` bez admin session je hotový,
  - ruční admin akce pro vydání krátkodobého app runtime JWT pro instalovanou
    aplikaci je hotová,
  - doplnit bezpečné předání/obnovu app runtime tokenu do Core-managed
    aplikačního runtime bez ručního kopírování,
  - doplnit feed/author podepsané update signály pro zdroje mimo instalovanou
    app runtime identitu,
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
