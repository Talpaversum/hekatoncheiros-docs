# Application runtime health

Installation and runtime availability are separate platform states. An
installed application remains registered during an outage, while its runtime
state starts as `unknown` after every Core restart and becomes `healthy`,
`degraded`, or `unreachable` only after health checks.

## Application contract

Applications expose unauthenticated `GET /health` on the internal platform
network. A successful response uses `healthy` or `degraded`:

```json
{"status":"healthy","service":"hc-app-inventory","version":"0.1.0","timestamp":"2026-07-15T14:00:00Z","checks":{"database":"healthy"}}
```

The manifest may declare `runtime.healthCheck.path`, `timeoutSeconds`, and
`intervalSeconds`. Older manifests use `/health`; failure never implies
healthy availability.

## Core behavior

Core checks enabled installed applications independently. Defaults are a
5-second interval, 1.5-second timeout, and two consecutive failures before
`unreachable`, keeping stale healthy status to roughly ten seconds. Platform
administrators can change all three values under Platform Configuration; the
environment variables `APP_RUNTIME_HEALTH_INTERVAL_MS`,
`APP_RUNTIME_HEALTH_TIMEOUT_MS`, and `APP_RUNTIME_HEALTH_FAILURE_THRESHOLD`
are startup fallbacks. A successful check resets failures.

`GET /api/v1/apps/registry` includes the safe runtime status, timestamps, and
failure count. Core logs state transitions structurally. The application proxy
allows `healthy` and `degraded`; other states return an
`application/problem+json` 503 without internal addresses or exception text.

## Web behavior

The registry and Installed Apps status are polled every five seconds. Unavailable navigation entries are
visible but disabled, direct application URLs show a platform availability
screen, and application plugins are not loaded until the runtime is usable.
The configurable dashboard includes Application runtime status.

## Troubleshooting

If an installed application stays unavailable, verify its container, internal
base URL, `/health` response, database dependency, and shared Docker network.
Use Core structured logs for transitions; do not expose internal health errors
to ordinary users.
