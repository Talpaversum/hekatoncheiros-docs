# API – hekatoncheiros-core (Licensing)

This document defines required core-side licensing API contracts for the target architecture.

## `POST /api/v1/tenants/{tenantId}/licenses/import`

Imports license material for a tenant.

### Accepted payload variants

Variant A (direct JWS):

```json
{
  "license_jws": "<...>",
  "author_cert_jws": "<optional ...>"
}
```

Variant B (bundle):

```json
{
  "bundle": {
    "bundle_typ": "hc-license-bundle",
    "v": 1,
    "license_jws": "<...>",
    "author_cert_jws": "<...>",
    "root_kid": "root_2026_01"
  }
}
```

### Response (example)

```json
{
  "status": "accepted",
  "license_jti": "lic_01J...",
  "selected": false,
  "validation": {
    "chain_verified": true,
    "mode": "portable"
  }
}
```

## `POST /api/v1/tenants/{tenantId}/licenses/validate`

Validates stored or provided license against local trust/policy state.

Request example:

```json
{
  "license_jti": "lic_01J..."
}
```

Response example:

```json
{
  "valid": true,
  "reason": null,
  "evaluated_at": "2026-02-17T10:30:00Z"
}
```

## OAuth activation endpoints

### `GET /api/v1/licenses/oauth/start`

Starts OAuth authorization (redirect flow).

### `GET /api/v1/licenses/oauth/callback`

Handles provider callback (`code`, `state`), exchanges token, and acquires license.

## `GET /api/v1/platform/instance-id`

Returns stable platform instance identifier.

Response example:

```json
{
  "platform_instance_id": "hcpi_550e8400-e29b-41d4-a716-446655440000"
}
```

## Selection API (recommended companion)

To support multi-license storage with explicit active choice:

- `POST /api/v1/tenants/{tenantId}/licenses/select`
- `POST /api/v1/tenants/{tenantId}/licenses/selection/clear`

These endpoints are companion recommendations for the required selection model.
