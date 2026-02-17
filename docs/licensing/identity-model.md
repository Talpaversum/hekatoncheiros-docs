# Identity Model

This document defines identities used by the licensing architecture.

## 1. Author identity (global)

- `author_id` is globally unique.
- Issued only by `hc-author-registry`.
- Used as canonical author namespace and license issuer identity.

Examples:

- `aut_01HVABCDEF...`

## 2. Author certificate identity binding

Author certificate is a root-signed JWS with:

- `sub = author_id`
- embedded `jwks` containing author signing keys

Certificate has no embedded status flag. Revocation is external.

## 3. License issuer identity

- License claim `iss` must equal `author_id`.
- `app.app_id` must start with the same `author_id/` prefix.

Validation rule:

`license.iss == prefix(app.app_id)`

## 4. Tenant identity (platform-local)

- `tenant_id` is local to a specific core platform instance.
- It is not global and not part of author namespace.
- Licenses are tenant-bound via `subject`.

Example:

- `tnt_123`

## 5. Platform instance identity

- Core has stable `platform_instance_id` (UUID-like identifier).
- Public endpoint: `GET /api/v1/platform/instance-id`
- Used in instance-bound audience checks (`hcpi_<uuid>`).

Policy note:

- If platform instance ID changes, instance-bound licenses become invalid.
- This behavior is accepted by design.
