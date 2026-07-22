# Docker Deployment

Status: works for local development and early self-hosted testing.

The Compose file lives in `hekatoncheiros-core/docker-compose.yml`.

It builds local images for:

- Core
- web shell

It runs:

- PostgreSQL
- Core API
- nginx serving the web shell and proxying `/api/v1/*` to Core
- an optional nginx HTTPS ingress with operator-supplied certificates

Author Registry and Licensing are separate products with separate databases
and Compose projects. They are not required by private self-hosted app
development.

## Local Author Registry

Start Core first, then initialize Registry and use its production Compose file:

```bash
cd ../hc-author-registry
npm ci
npm run pki:bootstrap:local
docker compose up -d --build
```

Registry uses port `4020`, joins the Core Compose network, and uses a dedicated
database reached through that network. Existing external `DATABASE_URL` values
are preserved. Generated `.local/pki` material and `.env` are ignored by Git.
The stack runs with production configuration validation, but its
`local-registry-root-*` identity is disposable local trust and must never be
promoted. Licensing issuer identity is provisioned separately through the
author workflow.

## Start

From `hekatoncheiros-core`:

```bash
docker compose up -d --build
docker compose run --rm core-seed
```

Open:

```text
http://localhost:8080
```

Default seeded login:

```text
admin@example.com / admin
```

## Useful URLs

- Web shell: `http://localhost:8080`
- Core API: `http://localhost:3000/api/v1`
- Core API through web proxy: `http://localhost:8080/api/v1`
- Swagger UI: `http://localhost:8080/docs`

## Configuration

Use a local `.env` next to `docker-compose.yml` for overrides.

Common values:

```env
WEB_PUBLISHED_PORT=8080
CORE_PUBLISHED_PORT=3000
POSTGRES_PUBLISHED_PORT=5432
POSTGRES_DB=hc_core
POSTGRES_USER=hc_user
POSTGRES_PASSWORD=replace-me
DATABASE_URL=postgres://hc_user:replace-me@postgres:5432/hc_core
JWT_SECRET=replace-with-generated-secret
INSTALLER_TOKEN_SECRET=replace-with-generated-secret
```

The built-in defaults are development-only.

## Optional HTTPS ingress

Store the certificate chain and private key outside the repository as
`tls.crt` and `tls.key`. Then set the public hostname and certificate directory:

```env
HTTPS_SERVER_NAME=hc.example.com
HTTPS_CERT_DIR=/secure/hekatoncheiros-certs
HTTPS_PUBLISHED_PORT=443
HTTP_REDIRECT_PUBLISHED_PORT=80
LICENSING_OAUTH_CALLBACK_BASE_URL=https://hc.example.com
WEB_PUBLISHED_PORT=127.0.0.1:8080
CORE_PUBLISHED_PORT=127.0.0.1:3000
POSTGRES_PUBLISHED_PORT=127.0.0.1:5432
```

Start Compose with the HTTPS override:

```bash
docker compose -f docker-compose.yml -f docker-compose.https.yml up -d --build
```

The ingress redirects HTTP to HTTPS, permits TLS 1.2 and 1.3, adds HSTS and
basic response hardening headers, and forwards the original client chain and
`X-Forwarded-Proto=https`. The certificate directory is mounted read-only.
Direct Core, web, and database ports must be loopback-bound as shown above or
blocked by the host firewall so external clients cannot bypass ingress.

## Stop

```bash
docker compose down
```

Remove volumes only when you intentionally want to delete local DB data:

```bash
docker compose down -v
```

## Current limitations

- external DB mode is not wired as an installer flow yet
- certificate acquisition and renewal remain operator responsibilities
- images are built locally, not published
- the verified Compose path is a development/local self-hosting deployment, not a packaged production installer
