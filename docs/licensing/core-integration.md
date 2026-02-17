# Core Integration Requirements

This document defines how `hekatoncheiros-core` integrates with the author-centric licensing model.

## Required endpoints

### Offline import

- `POST /api/v1/tenants/{tenantId}/licenses/import`

Accepts:

- `license_jws`
- optional `author_cert_jws`
- or full bundle payload

### Validation

- `POST /api/v1/tenants/{tenantId}/licenses/validate`

### OAuth activation

- `GET /api/v1/licenses/oauth/start`
- `GET /api/v1/licenses/oauth/callback`

### Platform instance ID

- `GET /api/v1/platform/instance-id`

## Mandatory validation logic (normative)

For each imported/activated license, core must execute:

1. Verify author certificate against trusted root JWKS.
2. Extract author JWKS from verified certificate payload.
3. Verify license JWS using extracted author key set.
4. Validate tenant binding and app namespace:
   - tenant scope required
   - `app_id` must be `<author_id>/<slug>`
   - `iss` must match `app_id` prefix
5. Validate audience policy:
   - `portable`: `aud` contains `"*"` (skip instance audience check)
   - `instance_bound`: `aud` must contain local `hcpi_<platform_instance_uuid>`

If any check fails, license is rejected.

## Storage and selection model

- Core stores all accepted licenses in `tenant_licenses`.
- Multiple licenses for same `(tenant_id, app_id)` are allowed.
- Core maintains exactly one selected active license per `(tenant_id, app_id)` in selection mapping.

Selection implications:

- New import does not auto-delete old licenses.
- Operator/admin can switch selected license explicitly.
- If selected license becomes invalid/expired, fallback policy may select another valid stored license.

## Install/uninstall integration policy

Manifest requirement:

```yaml
licensing:
  required: true | false
```

- `required=true`: install/enable blocked without valid selectable license.
- `required=false`: install allowed; feature enforcement happens at runtime.

Uninstall behavior:

- active licenses are archived (not deleted)
- expired licenses may be purged by retention policy
- history should remain auditable

## Offline behavior notes

- Runtime verification must work without network access.
- Revocation propagation is best-effort/manual in offline mode.
- Expiration (`exp`) is primary offline enforcement mechanism.
