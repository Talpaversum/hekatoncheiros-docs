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
- **Licensing**: entitlements (online + offline), selection, deterministic resolver
- **Apps**: registry + enable/disable per tenant
- **Events**: publish/consume/ack (app identity)
- **Audit**: append-only audit records

## Apps API behavior (entitlements model)

- `GET /api/v1/apps/registry`
  - source: globally installed apps (`core.installed_apps`)
  - visibility filter: `installed ∩ resolved entitlement for tenant ∩ privilege checks`
  - intended use: Apps dropdown / runtime discovery in web shell

- `GET /api/v1/apps/installed`
  - source: all globally installed apps (`core.installed_apps`)
  - includes tenant-scoped fields:
    - `resolved_entitlement`
    - `has_any_entitlement`
  - intended use: Manage apps (`/admin/apps`)
  - access: platform app management privilege required

- `GET /api/v1/apps/:slug/entitlement`
  - returns resolved entitlement for runtime (`tier`, `limits`, validity window, source)
  - `204` if no resolved entitlement

## Licensing endpoints (entitlements)

- `GET /api/v1/licensing/entitlements?app_id=...`
- `POST /api/v1/licensing/selection`
- `POST /api/v1/licensing/selection/clear`
- `POST /api/v1/licensing/offline`

Offline token requirements:
- JWS/JWT signature verification against configured keyring
- `aud` must match platform instance id
- required claims: `iss`, `aud`, `kid`, `tenant_id`, `app_id`, `tier`, `valid_from`, `valid_to`, `jti`
- optional claim: `limits`

## Links

- OpenAPI: [`openapi.yml`](./openapi.yml)
- Licensing architecture docs: [`../../licensing/overview.md`](../../licensing/overview.md)
