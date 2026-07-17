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

## Development Registry and Licensing

Generate disposable development-only trust material and start both services:

```bash
cd ../hc-author-registry
npm run pki:bootstrap:dev
docker compose --env-file .local/dev-pki/registry.env up -d --build

cd ../hc-app-licensing
docker compose --env-file ../hc-author-registry/.local/dev-pki/issuer.env up -d --build

cd ../hekatoncheiros-core
docker compose -f docker-compose.yml -f docker-compose.dev-trust.yml up -d --build
```

The Registry uses ports `4020` and `5435`; Licensing uses `4030` and `5434`.
Both join the Core Compose network for local back-channel traffic. The
generated `.local/dev-pki` material is ignored by Git and is not a production
trust anchor. Production mode never generates a Registry root automatically.
The `docker-compose.dev-trust.yml` override contains explicitly labeled,
disposable Core development identities and permits HTTP only for the local
Registry network. Do not include this override in production.

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

- app backend containers are not generated yet
- external DB mode is not wired as an installer flow yet
- certificate acquisition and renewal remain operator responsibilities
- images are built locally, not published
