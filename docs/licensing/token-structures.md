# Token Structures

This document defines canonical token and bundle payload structures.

## 1) Author Certificate (JWS, root-signed)

Payload:

```json
{
  "typ": "hc-author-cert",
  "v": 1,
  "iss": "hc-author-registry",
  "sub": "aut_01HVABCDEF...",
  "iat": 1760000000,
  "nbf": 1760000000,
  "exp": 1767777777,
  "jwks": {
    "keys": [
      {
        "kty": "EC",
        "crv": "P-256",
        "kid": "k1_2026_01",
        "x": "...",
        "y": "..."
      }
    ]
  }
}
```

Notes:

- `sub` is `author_id`.
- Certificate contains embedded author JWKS.
- No `status` field; revocation is external.

## 2) License Token (Author-signed JWS)

### 2.1 Portable license

```json
{
  "typ": "hc-license",
  "v": 1,
  "iss": "aut_01HVABCDEF...",
  "jti": "lic_01J...",
  "iat": 1760000000,
  "nbf": 1760000000,
  "exp": 1765000000,

  "subject": { "scope_type": "tenant", "tenant_id": "tnt_123" },
  "app": { "app_id": "aut_01HVABCDEF.../inventory" },

  "license_mode": "portable",
  "aud": ["*"],

  "features": { "rbac_advanced": false },
  "limits": { "seats": 25 }
}
```

Portable audience rule:

- if `aud` contains `"*"`, audience check is skipped.

### 2.2 Instance-bound license

```json
{
  "license_mode": "instance_bound",
  "aud": ["hcpi_<platform_instance_uuid>"]
}
```

Instance-bound audience rule:

- `aud` must contain local platform instance ID in `hcpi_` format.

## 3) Required semantic constraints

1. `iss` must equal `author_id`.
2. `app.app_id` must be prefixed by `author_id/`.
3. Prefix in `app.app_id` must match `iss`.
4. `subject.scope_type` must be `tenant`.
5. `jti` must be unique per issued license.

## 4) Offline bundle format

```json
{
  "bundle_typ": "hc-license-bundle",
  "v": 1,
  "license_jws": "<...>",
  "author_cert_jws": "<...>",
  "root_kid": "root_2026_01"
}
```

Bundle use:

- allows offline import where core may not already have author certificate cached.
