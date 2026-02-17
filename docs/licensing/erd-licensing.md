# ERD – hc-licensing

This document defines logical data model for author-hostable licensing service.

## `license_grants`

Purpose: issued licenses/grants ledger.

Fields:

- `id` (PK, uuid)
- `jti` (text, unique)
- `author_id` (text)
- `tenant_id` (text)
- `platform_instance_id` (text)
- `app_id` (text)
- `license_mode` (text: `portable|instance_bound`)
- `license_jws` (text)
- `issued_at` (timestamptz)
- `not_before` (timestamptz)
- `expires_at` (timestamptz)
- `status` (text: `active|revoked|expired`)
- `revoked_at` (timestamptz, nullable)

Constraints & indexes:

- PK (`id`)
- unique (`jti`)
- index (`tenant_id`, `app_id`, `status`)
- index (`platform_instance_id`)
- index (`expires_at`)

Lifecycle:

- created on issue/renewal
- renewal creates a new row/new `jti`
- revocation marks status, does not delete row

## `oauth_clients`

Purpose: registered OAuth clients (core instances).

Fields:

- `id` (PK, uuid)
- `client_id` (text, unique)
- `client_secret_hash` (text, nullable for public clients)
- `platform_instance_id` (text)
- `client_name` (text)
- `redirect_uris` (jsonb)
- `software_statement_iss` (text)
- `created_at` (timestamptz)
- `status` (text: `active|disabled`)

Constraints & indexes:

- unique (`client_id`)
- unique (`platform_instance_id`, `client_name`) optional policy
- index (`platform_instance_id`)

Lifecycle:

- inserted by DCR endpoint
- can be disabled/re-registered

## `oauth_tokens`

Purpose: issued OAuth tokens and optional refresh state.

Fields:

- `id` (PK, uuid)
- `client_id` (FK -> `oauth_clients.client_id`)
- `subject_id` (text; tenant admin identity on licensing server)
- `tenant_id` (text)
- `scope` (text)
- `access_token_hash` (text)
- `refresh_token_hash` (text, nullable)
- `issued_at` (timestamptz)
- `expires_at` (timestamptz)
- `revoked_at` (timestamptz, nullable)

Constraints & indexes:

- FK (`client_id`) -> `oauth_clients.client_id`
- index (`tenant_id`, `client_id`)
- index (`expires_at`)
- index (`revoked_at`)

Lifecycle:

- created during token exchange
- revoked/expired rows retained for audit
