# Core API (reference)

This document is a **conceptual overview** of the Core API. The detailed and
authoritative specification lives in the OpenAPI file:

- [`openapi.yml`](./openapi.yml)

## API principles

- All endpoints are versioned: `/api/v1/...`.
- Every request enforces a `RequestContext`:
  - `actor`: real user, effective user (impersonation), delegation
  - `tenant`: tenant id + tenancy mode
- Apps never receive raw platform auth tokens (context is injected via gateway or signed token).

## Main domains

- **Auth & session**: login, refresh, logout
- **Context**: `GET /api/v1/context`
- **Tenants**: tenant management (platform admin / install-time)
- **Users**: core-owned identity, read-only for apps
- **Access**: privileges, delegations, impersonation
- **Licensing**: license state and offline activation
- **Apps**: registry + enable/disable per tenant
- **Events**: publish/consume/ack (app identity)
- **Audit**: append-only audit records

## Links

- OpenAPI: [`openapi.yml`](./openapi.yml)
