# ERD – hc-author-registry

This document defines logical data model for author identity authority.

## `authors`

Purpose: global author registry.

Fields:

- `author_id` (PK, text)
- `display_name` (text)
- `status` (text, e.g. `active|suspended`)
- `created_at` (timestamptz)
- `updated_at` (timestamptz)

Constraints & indexes:

- PK: (`author_id`)
- unique index on normalized author slug/name if applicable
- index on `status`

Lifecycle:

- created once by registry onboarding
- may be suspended/revoked operationally

## `author_keys`

Purpose: author signing key records.

Fields:

- `id` (PK, uuid)
- `author_id` (FK -> `authors.author_id`)
- `kid` (text)
- `kty` (text)
- `crv` (text, nullable for non-EC)
- `public_jwk` (jsonb)
- `private_key_ref` (text, secret manager reference)
- `status` (text: `active|retired|revoked`)
- `created_at` (timestamptz)
- `revoked_at` (timestamptz, nullable)

Constraints & indexes:

- unique (`author_id`, `kid`)
- FK (`author_id`) -> `authors.author_id`
- index on (`author_id`, `status`)

Lifecycle:

- key rotation creates new row with new `kid`
- compromised key moved to `revoked`

## `author_certs`

Purpose: issued author certificate history.

Fields:

- `id` (PK, uuid)
- `author_id` (FK -> `authors.author_id`)
- `cert_jws` (text)
- `root_kid` (text)
- `issued_at` (timestamptz)
- `not_before` (timestamptz)
- `expires_at` (timestamptz)

Constraints & indexes:

- FK (`author_id`) -> `authors.author_id`
- index on (`author_id`, `expires_at`)
- index on `root_kid`

Lifecycle:

- new cert issued when keyset changes or renewals occur
- cert remains verifiable until `expires_at`
