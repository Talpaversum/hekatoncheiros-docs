# Licensing Architecture Overview

Status: **Target architecture (pre-alpha, documentation-first)**

This section defines a multi-instance, author-centric licensing model for Hekatoncheiros.

## Goals

- Global author identity via central `hc-author-registry`
- Author-issued licenses (central or self-hosted licensing instance)
- Two license policies: `portable` and `instance_bound`
- Online activation (OAuth2) and offline import
- Offline-capable cryptographic verification chain:
  - Root key -> Author certificate -> License JWS

## Components

1. **hc-author-registry (private authority)**
   - issues globally unique `author_id`
   - issues root-signed author certificates
   - publishes root JWKS and revocation snapshots

2. **hc-licensing (public, author-hostable)**
   - OAuth2 authorization code flow
   - Dynamic Client Registration (DCR)
   - license issuing and optional revocation
   - offline bundle export

3. **hekatoncheiros-core**
   - stores tenant licenses
   - validates certificate/license chain offline
   - provides import/validate/OAuth integration APIs
   - supports one selected active license per `(tenant_id, app_id)`

## Scope and policy decisions

- Licenses are always **tenant-bound**.
- No platform-scope license mode.
- `app_id` must use namespace format: `<author_id>/<slug>`.
- Hard cutover from legacy app IDs is expected (migration handled separately).

## Reading order

1. [Trust model](./trust-model.md)
2. [Identity model](./identity-model.md)
3. [Token structures](./token-structures.md)
4. [OAuth flow](./oauth-flow.md)
5. [Offline flow](./offline-flow.md)
6. [Core integration](./core-integration.md)
7. API and ERD documents
