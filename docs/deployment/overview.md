# Deployment overview

Status: draft for development and early self-hosted installations.

This section defines deployment procedures for Hekatoncheiros (HC) in three
operational modes:

- Baremetal / VM
- Docker / Docker Compose
- Kubernetes

The current implementation is still pre-production. Deployment material must
therefore distinguish between:

- **MVP now**: what can be run with the current repositories.
- **Target production**: the intended operational model once installer,
  packaging, artifact validation, and lifecycle automation are complete.

## Deployment goals

- Allow a self-hosted HC instance to run on one server, a VM, Docker, or a
  Kubernetes cluster.
- Keep the Core as the owner of identity, tenancy, app lifecycle, licensing,
  audit, and UI plugin runtime URLs.
- Support local application installation from official sources or uploaded
  application packages.
- Support externally operated applications as a separate integration mode.
- Keep database location configurable in every deployment mode.
- Start with HTTP-only deployments during development.
- Add HTTPS later through a reverse proxy or ingress layer.

## Baseline components

The base HC deployment contains:

- PostgreSQL
- `hekatoncheiros-core`
- `hekatoncheiros-web`

Application deployment is described generically. A concrete app may add:

- an app backend service
- app database schema or database access credentials
- a UI plugin artifact
- internal installer endpoints
- migrations or migration orchestration

## Default ports in development

- Core API: `3000`
- Web shell dev server: `5173`
- Example app backend: `4010`
- Author registry: `4020`
- Licensing server: `4030`

Production deployments should not rely on these public ports directly. They
should place a reverse proxy, gateway, or ingress in front of the services.

## URL model

The default recommendation is one canonical HC base URL per instance, for
example:

```text
http://hc.example.com
```

The reverse proxy or ingress routes paths to internal services:

- `/api/v1/*` -> Core API
- `/app/*` -> web shell routes rendered by the platform shell
- Core-owned UI plugin URLs -> Core API/static artifact serving

Custom application hostnames should be supported later as aliases, for example:

```text
http://inv.example.com
```

Such hostnames should still resolve into the same HC instance and should not
make the application own authentication, routing, or UI execution. A possible
future model is:

- `inv.example.com` as CNAME or DNS alias to the HC ingress address
- host-based routing maps the request to the same web shell
- Core resolves the requested host to an app route or tenant context

This requires explicit design before production use because it affects tenant
resolution, app routing, cookies, CORS, trusted origins, and audit context.

## TLS model

During development the default deployment is HTTP-only.

HTTPS is expected to be added later at the reverse proxy or ingress boundary,
for example:

- Traefik
- nginx
- Caddy
- Kubernetes Ingress Controller
- cloud load balancer

Internal traffic behind the proxy may stay HTTP in development. Production
hardening must revisit this, especially for installer channels and internal app
endpoints.

## Database placement

Every deployment mode supports two database placement choices:

- **Managed by deployment**: the deployment creates PostgreSQL locally, in
  Docker, or in Kubernetes.
- **External database**: the operator provides database credentials during
  installation and the deployment does not create PostgreSQL.

Defaults:

- Baremetal / VM: operator provides PostgreSQL and credentials.
- Docker: Docker Compose creates PostgreSQL unless external DB is selected.
- Kubernetes: Kubernetes manifests create PostgreSQL unless external DB is
  selected.

## Application installation modes

HC must support these application inclusion modes:

- **Official source install**: HC installs an app from an official source.
- **Local package upload**: an operator uploads app files to the HC instance;
  Core validates them and installs them with the same lifecycle as official
  source installs.
- **Standalone app integration**: the app is operated separately and is not
  started or managed by the target HC instance.

For Core-managed app installations, the target direction is:

- Core validates manifest and package metadata.
- Core builds or installs runtime artifacts according to the deployment mode.
- Core stores and serves UI plugin artifacts through Core-owned URLs.
- Core controls app lifecycle state.

During development it is acceptable to build container images locally during
installation. Images are not published to Docker Hub or another registry by
default.

## Open architectural issue: standalone apps and database ownership

Standalone application integration is unresolved.

The tension:

- A standalone app may need its own database lifecycle and operational control.
- HC requires apps to work against the target HC instance's tenant/app data
  model and isolation rules.
- Apps must not access Core schema directly.
- Apps should not create databases in the target HC instance.

This affects:

- DB credentials
- schema ownership
- migration authority
- backup and restore boundaries
- network policy
- app disablement and uninstall semantics

Until resolved, deployment documentation must treat standalone apps as a
separate future integration mode and avoid presenting it as production-ready.

See also the TODO item in `docs/ROADMAP.md`.

## Configuration and secrets

Recommended baseline:

- Baremetal / VM: environment file owned by the service user, permissions
  `0600`.
- Docker: `.env` next to `docker-compose.yml`, not committed to git.
- Kubernetes: `Secret` for sensitive values and `ConfigMap` for non-sensitive
  configuration.

Secrets must never be logged and must not be embedded in generated static web
assets.

