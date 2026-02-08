# Core API (reference)

This document is a **conceptual overview** of the Core API. The detailed and
authoritative specification lives in the OpenAPI file:

- [`openapi.yml`](./openapi.yml)

## API principles

- All endpoints are versioned: `/api/v1/...`.
- Every request enforces a `RequestContext`:
  - `actor`: real user, effective user (impersonation), delegation
  - `tenant`: tenant id + tenancy mode
- Apps never receive raw platform auth tokens (context is injected via gateway or signed token).

## Main domains

- **Auth & session**: login, refresh, logout
- **Context**: `GET /api/v1/context`
- **Tenants**: tenant management (platform admin / install-time)
- **Users**: core-owned identity, read-only for apps
- **Access**: privileges, delegations, impersonation
- **Licensing**: license state and offline activation
- **Apps**: registry + enable/disable per tenant
- **Events**: publish/consume/ack (app identity)
- **Audit**: append-only audit records

## Apps API behavior (current)

- `GET /api/v1/apps/registry`
  - source: globally installed apps (`core.installed_apps`)
  - visibility filter: `installed ∩ active license for tenant ∩ privilege checks`
  - intended use: Apps dropdown / runtime discovery in web shell

- `GET /api/v1/apps/installed`
  - source: all globally installed apps (`core.installed_apps`)
  - includes tenant-scoped field `licensed: boolean`
  - intended use: Manage apps (`/admin/apps`)
  - access: platform app management privilege required

## Links

- OpenAPI: [`openapi.yml`](./openapi.yml)
