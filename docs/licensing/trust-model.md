# Trust Model

This document defines the root of trust and verification chain for licensing.

Development and test roots are disposable, explicitly labeled trust anchors.
They are never production roots and production services must reject them. See
[Licensing PKI Bootstrap](./pki-bootstrap.md) for the implemented local workflow
and the deferred production ceremony.

## Root of trust

- Trust anchor is managed by `hc-author-registry`.
- `hc-author-registry` publishes root public keys at:
  - `GET /v1/root/jwks`
- Core trusts only root keys pinned in its configuration/release.

## Verification chain

Mandatory chain:

1. Verify **Author Certificate JWS** with trusted root key (`kid` in root JWKS)
2. Extract author JWKS from certificate payload
3. Verify **License JWS** with key from extracted author JWKS
4. Apply policy checks (tenant scope, app namespace, audience mode, time window)

If any step fails, license is invalid.

## Root key rotation strategy

### Publishing model

- Root JWKS can contain multiple root keys at the same time.
- Each root key has unique `kid`.

### Rotation behavior

- New root key is added.
- Old root key stays published during overlap period (recommended 6-12 months).
- New author certs are signed with new root key.
- Existing certs signed by old key remain valid until `exp`.

### Core behavior

- Core can operate fully offline with pinned root keys.
- Online JWKS fetch is for key update workflows, not runtime requirement.

## Revocation model

Revocation data endpoint:

- `GET /v1/revocations`

Structured snapshot:

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

## Offline limitations (explicit)

- Offline environments are not guaranteed to receive revocations.
- Offline mode may rely only on expiration (`exp`) if snapshot import is not performed.
- Central `jti` revocation propagation is out of scope for now.
- Core may keep local denylist (`license_jti`) as platform-local policy.
