# Baremetal / VM deployment

Status: draft. This is the intended operator flow for a single machine or VM.

## MVP now

The current repositories run as Node.js services. A baremetal deployment should
install and run at least:

- PostgreSQL
- Node.js runtime compatible with the repositories
- `hekatoncheiros-core`
- `hekatoncheiros-web` build output served by a web server or reverse proxy

PostgreSQL credentials are provided by the operator during installation.

## Recommended system layout

Example paths:

```text
/opt/hekatoncheiros/core
/opt/hekatoncheiros/web
/opt/hekatoncheiros/apps
/var/lib/hekatoncheiros/core-data
/etc/hekatoncheiros/core.env
```

The service user should own runtime data and read config files:

```text
hc:hc
```

Sensitive env files should use permissions:

```text
0600
```

## Core configuration

Minimum Core values:

```env
PORT=3000
DATABASE_URL=postgres://hc_user:hc_password@127.0.0.1:5432/hc_core
CORE_DATA_DIR=/var/lib/hekatoncheiros/core-data
TENANCY_MODE=row_level
JWT_ISSUER=hekatoncheiros-core
JWT_AUDIENCE_USER=hc-user
JWT_AUDIENCE_APP=hc-app
JWT_SECRET=replace-with-generated-secret
INSTALLER_TOKEN_SECRET=replace-with-generated-secret
INSTALLER_TOKEN_ISSUER=hekatoncheiros-core-installer
DEFAULT_TENANT_ID=tnt_default
LICENSING_OAUTH_CALLBACK_BASE_URL=http://hc.example.com
```

## Installation flow

1. Install PostgreSQL or prepare external database credentials.
2. Create database and runtime user if they do not exist.
3. Clone or unpack `hekatoncheiros-core`.
4. Install dependencies.
5. Build Core.
6. Run Core migrations.
7. Seed initial data.
8. Configure Core as a system service.
9. Build `hekatoncheiros-web`.
10. Serve the web build through a reverse proxy.
11. Route `/api/v1/*` to Core.
12. Run health checks.

## Commands: current development repositories

Core:

```bash
npm install
npm run build
npm run db:migrate
npm run db:seed
npm run start
```

Web:

```bash
npm install
npm run build
```

## Reverse proxy

During development HTTP is acceptable.

The proxy should route:

- `/api/v1/*` to `http://127.0.0.1:3000`
- all web routes to the web shell static build

HTTPS termination can be added later at the proxy.

## App installation

For Core-managed apps, baremetal installation may build and run app processes
locally during development.

A future installer should:

- validate uploaded or downloaded app package metadata
- build required local artifacts if allowed by the current deployment policy
- create app runtime config
- register app backend URL with Core
- fetch and store the UI plugin artifact through Core
- move app lifecycle to `ready`

Standalone apps are not production-defined yet. See deployment overview.

