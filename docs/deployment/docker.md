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
- HTTPS is out of scope for now
- images are built locally, not published
