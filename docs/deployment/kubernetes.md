# Kubernetes deployment

Status: draft. Kubernetes support is a target deployment mode, not yet backed
by manifests or a Helm chart.

## MVP target

The first Kubernetes deployment should include:

- Namespace
- PostgreSQL StatefulSet or external database configuration
- Core Deployment
- Web Deployment
- Service objects
- Ingress
- Secret and ConfigMap
- PersistentVolumeClaim for Core-owned data

Application deployment should be generic and app-driven, not hardcoded to a
specific app.

## Database options

Kubernetes deployment supports two modes:

- **In-cluster DB**: PostgreSQL runs as a StatefulSet with PVC.
- **External DB**: operator provides credentials in a Kubernetes Secret.

In-cluster DB is acceptable for development and simple self-hosted installs.
Production should strongly consider external managed PostgreSQL or a dedicated
PostgreSQL operator.

## Configuration

Use:

- `ConfigMap` for non-sensitive values
- `Secret` for passwords, tokens, private keys, and DSNs containing passwords

Example secret keys:

```text
DATABASE_URL
JWT_SECRET
INSTALLER_TOKEN_SECRET
LICENSING_ROOT_JWKS_JSON
```

Example config keys:

```text
PORT
CORE_DATA_DIR
TENANCY_MODE
JWT_ISSUER
JWT_AUDIENCE_USER
JWT_AUDIENCE_APP
DEFAULT_TENANT_ID
LICENSING_OAUTH_CALLBACK_BASE_URL
```

## Ingress and hostnames

The default recommendation is one ingress host:

```text
hc.example.com
```

Path routing:

- `/api/v1/*` -> Core Service
- web routes -> Web Service

Custom app hostnames are future work. They must route into the same HC instance
and preserve Core-owned authentication, tenant resolution, and audit context.

## TLS

Development Kubernetes deployments may use HTTP.

HTTPS should be added later through ingress TLS termination, for example:

- Traefik
- nginx ingress
- cert-manager
- cloud load balancer certificates

## Probes

Core:

```text
readiness: /api/v1/readyz
liveness: /api/v1/healthz
```

App backends:

```text
readiness: /health
liveness: /health
```

## App deployment model

For Core-managed apps, Kubernetes installation may initially build images
locally before deployment. This is acceptable for development only.

Target production should use prebuilt and verified artifacts.

The generic app deployment must support:

- app backend Deployment
- app Service internal to the namespace
- env from Secret and ConfigMap
- optional PVC if the app needs local storage
- Core-to-app internal installer channel
- network policy to restrict installer-only endpoints where possible

Generated manifests must use app metadata such as slug, declared ports, and
required environment values. They must not hardcode a specific application.

Standalone app integration remains unresolved and must not be presented as a
production-ready mode.

## TODO: Helm or Kustomize

Choose one packaging strategy:

- Helm chart for operator-friendly install-time options
- Kustomize overlays for simpler generated manifests

The initial recommendation is Helm once configuration options grow beyond a
small static manifest set.
