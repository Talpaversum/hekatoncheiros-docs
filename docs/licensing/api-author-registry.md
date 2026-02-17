# API – hc-author-registry

This document describes external contracts relevant for licensing trust.

## `GET /v1/root/jwks`

Returns active root public keys.

### 200 OK (example)

```json
{
  "keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "kid": "root_2026_01",
      "x": "...",
      "y": "..."
    },
    {
      "kty": "EC",
      "crv": "P-256",
      "kid": "root_2025_01",
      "x": "...",
      "y": "..."
    }
  ]
}
```

Notes:

- Multiple root keys may coexist during rotation overlap.
- Core runtime does not require live fetch if keys are pinned locally.

## `GET /v1/revocations`

Returns structured revocation snapshot.

### 200 OK (canonical structure)

```json
{
  "version": 1,
  "updated_at": "2026-02-17T10:00:00Z",
  "revoked_author_kids": [
    {
      "author_id": "aut_01HVABCDEF...",
      "kid": "k1_2026_01",
      "revoked_at": "2026-02-10T08:12:00Z",
      "reason": "key_compromise"
    }
  ],
  "revoked_author_ids": [],
  "revoked_root_kids": []
}
```

### Field semantics

- `revoked_author_kids`: key-level revocations for author signing keys
- `revoked_author_ids`: author-level emergency revocations
- `revoked_root_kids`: root key revocations/disallow list
- `reason`: informational, not enforcement-critical

## Internal/private operations

Author registration and author certificate issuing endpoints are internal/private to registry operations and are not specified as public contracts here.
