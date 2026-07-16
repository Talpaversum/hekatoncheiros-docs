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

### Offline activation request

- `POST /api/v1/licensing/offline/request`

Returns a short-lived Core-signed `hc-license-activation-request` and the Core
public signing key. The issuer verifies the signature, registered instance, and
an active offline-enabled grant before accepting the request.

### Revocation synchronization

- `POST /api/v1/platform/author-registry/sync-trust`
- `POST /api/v1/licensing/revocations/sync`

Core stores the last successful registry and issuer snapshots. Verification
continues offline from these snapshots.

## Mandatory validation logic (normative)

For each imported/activated license, core must execute:

1. Verify author certificate against the explicitly trusted registry root JWKS
   and registry identifier.
2. Extract author JWKS from verified certificate payload.
3. Verify license JWS using extracted author key set.
4. Validate tenant binding and app namespace:
   - tenant scope required
   - `app_id` must be `<author_id>/<slug>`
   - `iss` must match `app_id` prefix
5. Reject revoked root keys, authors, author keys, and license JTIs using the
   latest stored snapshots.
6. Validate audience policy:
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

- `required=true`: install is allowed without a license, but tenant runtime use is blocked until the tenant has a selected active license for the app.
- `required=false`: install and runtime use are allowed without a license; optional feature enforcement may still happen inside the app.

Installation is technical provisioning: manifest validation, artifact fetch/storage,
proxy metadata, and app lifecycle records. Licensing gates activation/use, not the
ability to stage an application into the instance.

Core must enforce the runtime boundary for license-required apps:

- hide the app from tenant runtime navigation/catalog responses unless a selected active license exists
- reject proxied app API calls without a selected active license
- expose entitlement state so the app can apply its own non-destructive feature and limit policy

Uninstall behavior:

- active licenses are archived (not deleted)
- expired licenses may be purged by retention policy
- history should remain auditable

## Offline behavior notes

- Runtime verification must work without network access.
- Revocation propagation is best-effort/manual in offline mode.
- Expiration (`exp`) is primary offline enforcement mechanism.
