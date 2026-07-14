# Baremetal / VM Deployment

Status: reference deployment available; operator automation is still external.

The reference layout uses operator-managed PostgreSQL, Node.js 22, built Core
and web distributions, systemd, and nginx HTTPS termination.

## Layout

```text
/opt/hekatoncheiros/core       built Core repository
/opt/hekatoncheiros/web        built web assets
/var/lib/hekatoncheiros        writable Core data
/etc/hekatoncheiros/core.env   mode 0600 configuration
/etc/hekatoncheiros/tls        operator-managed certificate and key
```

Run Core as a dedicated `hekatoncheiros` user. Install the reference files
`deploy/systemd/hekatoncheiros-core.service` and
`deploy/nginx/baremetal.conf.example`, replacing hostname and TLS paths.

## Core configuration

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://hc_user:replace-me@db.example.com:5432/hc_core
CORE_DATA_DIR=/var/lib/hekatoncheiros/core-data
JWT_SECRET=replace-with-a-generated-secret
INSTALLER_TOKEN_SECRET=replace-with-a-different-generated-secret
LICENSING_OAUTH_CALLBACK_BASE_URL=https://hc.example.com
APP_RUNTIME_DOCKER_ENABLED=false
```

Keep `/etc/hekatoncheiros/core.env` mode 0600; never put secrets in the unit.

## Install and verify

1. Build Core with `npm ci && npm run build` and install it under `/opt`.
2. Build the web shell and copy its `dist` contents to the web path.
3. Install and enable the systemd unit.
4. Verify nginx with `nginx -t`, then reload it.
5. Start Core. Startup applies migrations under a PostgreSQL advisory lock.
6. Seed only a new installation under the same environment.

```bash
systemd-analyze verify /etc/systemd/system/hekatoncheiros-core.service
curl --fail http://127.0.0.1:3000/api/v1/readyz
curl --fail https://hc.example.com/api/v1/readyz
```

Only nginx ports 80/443 should be public. Back up PostgreSQL and Core data
according to application consistency requirements.
