# ERD – hekatoncheiros-core (Licensing)

This document defines target licensing tables in core.

## `tenant_licenses`

Purpose: stores all imported/activated tenant licenses.

Fields:

- `id` (PK, uuid)
- `tenant_id` (text)
- `app_id` (text)
- `license_jti` (text)
- `license_mode` (text: `portable|instance_bound`)
- `issuer_author_id` (text)
- `license_jws` (text)
- `author_cert_jws` (text, nullable)
- `valid_from` (timestamptz)
- `valid_to` (timestamptz)
- `status` (text: `active|archived|revoked|expired`)
- `source` (text: `offline|oauth`)
- `imported_at` (timestamptz)

Constraints & indexes:

- PK (`id`)
- unique (`tenant_id`, `license_jti`)
- index (`tenant_id`, `app_id`, `status`)
- index (`valid_to`)

Lifecycle:

- import/activation inserts new row
- license switch does not delete previous rows
- uninstall archives active rows for audit

## `oauth_connections`

Purpose: stores author licensing server connection and client registration metadata.

Fields:

- `id` (PK, uuid)
- `platform_instance_id` (text)
- `author_id` (text)
- `licensing_base_url` (text)
- `client_id` (text)
- `client_secret_ref` (text, nullable)
- `token_endpoint_auth_method` (text)
- `created_at` (timestamptz)
- `status` (text: `active|disabled`)

Constraints & indexes:

- unique (`platform_instance_id`, `author_id`, `licensing_base_url`)
- index (`author_id`)
- index (`status`)

Lifecycle:

- created by DCR bootstrap
- rotated or replaced on re-registration

## `license_revocations_local`

Purpose: local denylist and imported revocation data for offline-aware enforcement.

Fields:

- `id` (PK, uuid)
- `tenant_id` (text, nullable for global entries)
- `license_jti` (text, nullable)
- `author_id` (text, nullable)
- `author_kid` (text, nullable)
- `root_kid` (text, nullable)
- `reason` (text)
- `source` (text: `local|snapshot`)
- `revoked_at` (timestamptz)
- `ingested_at` (timestamptz)

Constraints & indexes:

- index (`tenant_id`, `license_jti`)
- index (`author_id`, `author_kid`)
- index (`root_kid`)

Lifecycle:

- populated by operator/local policy or revocation snapshot import
- never hard-deleted by default (auditability)

## `tenant_app_license_selection`

Purpose: one active selected license per tenant/app.

Fields:

- `tenant_id` (text)
- `app_id` (text)
- `license_jti` (text)
- `selected_at` (timestamptz)
- `selected_by` (text, nullable)

Constraints & indexes:

- PK (`tenant_id`, `app_id`)
- FK (`tenant_id`, `license_jti`) -> `tenant_licenses` logical relation
- index (`license_jti`)

Lifecycle:

- explicit set/clear by user or policy
- fallback resolver may be used when selection becomes invalid
