# Baremetal / VM Deployment

Status: sketch.

Baremetal deployment is not automated yet. For now, treat it as the same
runtime pieces as Docker, but installed directly on a host.

## Required pieces

- PostgreSQL provided by the operator
- Node.js runtime
- `hekatoncheiros-core`
- built `hekatoncheiros-web`
- reverse proxy serving the web shell and proxying `/api/v1/*` to Core

## Minimal flow

1. Prepare PostgreSQL and `DATABASE_URL`.
2. Build Core.
3. Run Core migrations and seed.
4. Run Core as a service.
5. Build the web shell.
6. Serve the web build through a reverse proxy.
7. Route `/api/v1/*` to Core.

## Notes

- Use an env file readable only by the service user.
- Start with HTTP during development.
- HTTPS termination belongs at the reverse proxy later.
- A proper installer should generate service files and hardened config.
