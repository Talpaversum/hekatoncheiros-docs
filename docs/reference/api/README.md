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
  - includes filtered `nav_entries` and optional procedural `help_entries`

- `GET /api/v1/apps/installed`
  - source: all globally installed apps (`core.installed_apps`)
  - includes tenant-scoped fields:
    - `resolved_entitlement`
    - `has_any_entitlement`
  - includes optional `catalog_update` when the local catalog has a matching
    `app_id`; this is a passive signal and does not mutate runtime state
  - `catalog_update.state` distinguishes `available`, `same`, `stale`, and
    `baseline_missing`
  - includes optional `update_signal` when an app, feed layer, or admin has
    reported that a newer manifest/UI artifact may be available
  - intended use: Manage apps (`/core/apps/installed`)
  - access: platform app management privilege required

- `DELETE /api/v1/apps/installed/{app_id}`
  - uninstalls the application from Core
  - for a Core-owned Compose runtime, removes the recorded service containers
    before deleting the installation and runtime ownership records
  - external application processes are never stopped by Core
  - access: platform app management privilege required; runtime management
    privilege is additionally required for Core-owned Compose runtimes

- `POST /api/v1/apps/installed/{app_id}/app-token`
  - admin action for issuing a short-lived app runtime JWT
  - token audience is the app API audience and the token contains `app_id`,
    `tenant_id`, and `purpose=core-api`
  - intended for development and manual operational flows until Core-managed
    runtime token delivery is implemented
  - access: platform app management privilege required

- `POST /api/v1/apps/installed/update-signal`
  - app-auth action for reporting that the calling installed app has a newer
    manifest/UI artifact available
  - Core takes `app_id` from the app JWT, not from request body or URL
  - stores `source=app` and does not mutate runtime state
  - intended follow-up action: admin reviews the signal and runs
    `refresh-artifact` or clears the signal

- `POST /api/v1/apps/installed/{app_id}/update-signal`
  - admin action for manually recording an update signal for an installed app
  - supports `source=app|feed|manual`
  - access: platform app management privilege required

- `DELETE /api/v1/apps/installed/{app_id}/update-signal`
  - admin action for clearing an active update signal without changing runtime
    state
  - access: platform app management privilege required

- `POST /api/v1/apps/installed/{app_id}/refresh-artifact`
  - admin action for refreshing an already installed app
  - Core fetches the current manifest from the installed app `base_url`
  - Core rejects the refresh if the fetched manifest `app_id` differs
  - Core downloads and stores the current UI plugin artifact and updates runtime
    metadata
  - clears any active `update_signal` for the app after a successful refresh
  - access: platform app management privilege required

- `POST /api/v1/apps/installed/{app_id}/check-update`
  - admin action for checking whether a refreshed manifest/UI artifact is
    available without mutating runtime state
  - Core fetches the current manifest from the installed app `base_url`
  - Core compares the fetched manifest hash with the stored installed manifest
    hash
  - returns `update_available=true|false|null`; `null` means the installed app
    does not yet have a stored baseline manifest hash
  - access: platform app management privilege required

- `POST /api/v1/apps/catalog/entries/{app_id}/refresh-from-installed`
  - admin action for refreshing or creating a catalog entry from an installed
    app's current `base_url`
  - Core fetches the installed app manifest and rejects mismatched `app_id`
  - existing catalog source/trust/summary/deployment metadata is preserved
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
