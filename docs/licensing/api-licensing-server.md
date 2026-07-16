# API – hc-licensing (author-hostable)

This document defines expected public contracts of author licensing servers.

## OAuth / DCR endpoints

### `POST /oauth/register`

Registers core client dynamically (per platform instance).

Request:

```json
{
  "software_statement": "<signed JWT>",
  "redirect_uris": ["https://core.example.com/api/v1/licenses/oauth/callback"],
  "client_name": "Hekatoncheiros Core"
}
```

`software_statement` required claims:

- `platform_instance_id`
- `iss`
- `iat`
- `exp`

Response (example):

```json
{
  "client_id": "cl_01J...",
  "client_secret": "...",
  "token_endpoint_auth_method": "client_secret_post"
}
```

### `GET /oauth/authorize`

Authorization Code flow authorization endpoint.

### `POST /oauth/token`

Token endpoint for authorization code exchange.

## Licensing endpoints

### `POST /v1/licenses/issue`

Issues license for authorized tenant/platform context.

Response (example):

```json
{
  "license_jws": "<...>",
  "author_cert_jws": "<...>",
  "issued_at": "2026-02-17T10:20:00Z"
}
```

### `GET /v1/licenses/bundle`

Returns offline bundle (`license_jws`, `author_cert_jws`, `root_kid`).

### `GET /v1/revocations`

Returns revoked license JTIs for Core snapshot synchronization.

## Administration API

The Core-hosted plugin uses delegated `/v1/admin/*` endpoints for products,
customers, instances, grants, activations, issued licenses, bundle download,
revocation, dashboard metrics, and audit. Each endpoint requires its matching
`licensing.*` privilege. The API has no local login or browser session.

## Behavioral requirements

- `iss` in license must be `author_id`.
- `app.app_id` must follow `<author_id>/<slug>`.
- license `jti` must be unique.
- Renewal issues new `jti` (no in-place rewrite of previous token).
