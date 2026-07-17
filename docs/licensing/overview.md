# Licensing Architecture Overview

Status: **Implemented pre-alpha foundation**

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

2. **hc-app-licensing (author-operated installable application)**
   - OAuth2 authorization code flow
   - Dynamic Client Registration (DCR)
   - license issuing and optional revocation
   - offline bundle export
   - products, customers, Core instances, commercial grants, activations, and
     issued licenses as separate lifecycle records
   - Core-hosted administration plugin using delegated identity and RBAC

3. **hekatoncheiros-core**
   - stores tenant licenses
   - validates certificate/license chain offline
   - provides import/validate/OAuth integration APIs
   - supports one selected active license per `(tenant_id, app_id)`
   - exports signed offline activation requests and stores the latest registry
     and issuer revocation snapshots

## Authentication boundary

Neither the issuer nor the registry owns users, passwords, login forms, or
browser sessions. Human administration uses a short-lived, application-bound
Ed25519 delegation JWS issued by Core. The token carries the actual and effective user,
tenant, privileges, correlation ID, and expiry. Each receiving service verifies
the token with a dedicated public JWKS and records its own attributable audit
event. Applications never receive Core's private delegation key or primary user
JWT secret.

Machine authentication is separate. The legacy shared `ADMIN_TOKEN` is a
development compatibility mechanism only; production integrations require
scoped, rotatable service identities.

Registry authority is established only by an explicitly trusted root key,
registry identifier, fingerprint, and trust-policy version. A compatible API or
hostname does not confer trust.

## Scope and policy decisions

- Licenses are always **tenant-bound**.
- No platform-scope license mode.
- `app_id` must use namespace format: `<author_id>/<slug>`.
- Hard cutover from legacy app IDs is expected (migration handled separately).

## Reading order

1. [Trust model](./trust-model.md)
2. [PKI bootstrap and environment separation](./pki-bootstrap.md)
3. [Identity model](./identity-model.md)
4. [Token structures](./token-structures.md)
5. [OAuth flow](./oauth-flow.md)
6. [Offline flow](./offline-flow.md)
7. [Core integration](./core-integration.md)
7. API and ERD documents
