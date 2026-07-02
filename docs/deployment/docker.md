# Docker deployment

Status: draft. Docker is the preferred development-friendly self-hosted mode.

## MVP now

The current Core repository contains a development `docker-compose.yml` for
PostgreSQL only. Production-like Docker deployment still needs Dockerfiles and
Compose definitions for:

- Core
- Web shell
- optional app backends
- optional PostgreSQL

During development, images may be built locally at installation time. Images are
not pushed to Docker Hub or another registry by default.

## Database options

Docker deployment supports two modes:

- **Internal DB**: Compose starts PostgreSQL.
- **External DB**: operator provides `DATABASE_URL`; Compose does not start
  PostgreSQL.

The installer or generated Compose profile should select one of these modes.

## Expected Compose services

Baseline internal DB deployment:

```text
postgres
core
web
```

Optional application services:

```text
app-<slug>
```

The app service is internal by default and should be generated from app
metadata. Deployment assets must not hardcode a specific application. User-
facing UI must still be rendered inside the HC web shell.

## Local image build policy

For now, Docker installation may build images from local repositories:

```bash
docker compose up -d --build
docker compose run --rm core-seed
```

Target production may later switch to:

- prebuilt images
- signed images
- private registry
- marketplace-provided packages

## Environment file

Docker Compose should use `.env` next to `docker-compose.yml`.

The file must not be committed to git.

Example:

```env
HC_BASE_URL=http://hc.local
POSTGRES_DB=hc_core
POSTGRES_USER=hc_user
POSTGRES_PASSWORD=replace-with-generated-secret
DATABASE_URL=postgres://hc_user:replace-with-generated-secret@postgres:5432/hc_core
JWT_SECRET=replace-with-generated-secret
INSTALLER_TOKEN_SECRET=replace-with-generated-secret
```

## Runtime routing

For development, Compose may expose:

- web: `8080`
- core: internal only, or exposed on `3000` for debugging

Recommended public entrypoint:

```text
http://localhost:8080
```

The reverse proxy container routes:

- `/api/v1/*` -> `core:3000`
- web routes -> `web`

The current Compose deployment also exposes Core on `localhost:3000` for
debugging. The web shell should normally use the `localhost:8080` entrypoint.

## App installation

For Core-managed apps in Docker mode, the installer should be able to:

- build the app image locally
- create or update the app service definition
- provide app env values
- keep the app backend reachable from Core over the Compose network
- prevent direct user-facing access to installer-only endpoints

The UI plugin artifact source endpoint must be fetched by Core using installer
identity. The web shell must load only Core-owned runtime `ui_url` values.

## TODO: generated Compose

Create generated Compose assets for:

- baseline internal DB
- external DB
- one generic app backend template
- reverse proxy
- persistent volumes
- health checks
