# Operations

Status: draft operational checklist for all deployment modes.

## Health checks

Core:

```text
GET /api/v1/healthz
GET /api/v1/readyz
```

App backends:

```text
GET /health
```

## Startup order

1. PostgreSQL or external DB connectivity.
2. Core migrations.
3. Core service.
4. Web shell.
5. App backend services.
6. App installation or registration.

## Migrations

Core migrations run before Core starts serving normal traffic.

Application migration authority is still an open design issue for standalone
apps. Deployment procedures must not assume that standalone apps are fully
resolved.

For Core-managed apps, the target installer must record lifecycle states:

- `registered`
- `installing`
- `migrating`
- `ready`
- `failed`
- `disabled`

## Backups

Minimum backup scope:

- PostgreSQL database or tenant databases
- Core-owned data directory
- deployment configuration
- secrets, stored securely outside the application host

For DB-per-tenant mode, backups should be tenant-aware.

## Restore

Restore order:

1. Restore database.
2. Restore Core-owned data directory.
3. Restore configuration and secrets.
4. Start Core.
5. Validate health and readiness.
6. Validate installed app registry and UI plugin artifact availability.

## Logs

Logs must not include:

- database passwords
- JWT secrets
- installer token secrets
- private keys
- full license private material

## Development limitations

The current deployment model is not production-ready.

Known limitations:

- no finalized installer
- no production Dockerfiles
- no Kubernetes manifests or Helm chart yet
- HTTP-only development assumption
- standalone app database model unresolved
- app image publishing intentionally out of scope for now

