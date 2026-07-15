# AAA accounting and audit log

Hekatoncheiros extends the existing `core.audit_log`; it does not introduce a
second logging subsystem. Fastify logs remain technical operational output for
requests, startup, stack traces, and debugging. Audit events are durable
business, security, and administrative facts. Application stdout is not audit
data.

## Event model

Every row retains the original `tenant_id`, actor IDs, `action`, `object_ref`,
`metadata`, `created_at`, and UUID primary key. Structured fields add occurrence
and receipt time, scope, visibility, category, severity, outcome, actor type,
application and service identity, event and resource identity, correlation and
request IDs, client information, message, and schema version.

Scope is ownership and visibility is authorization; neither implies the other:

- `user`, `tenant`, and `platform` identify event ownership;
- `user`, `tenant_admin`, and `platform_admin` identify minimum read access;
- platform events have no tenant; user and tenant events require one.

Older calls to `recordAudit` are normalized into schema version 1. Stable event
types use lowercase dotted identifiers such as `auth.login.succeeded`,
`identity.user.created`, and `app.runtime.started`. Severity, outcome, category,
and event type are stored as technical identifiers. The English `message` is a
diagnostic fallback. Web translates known identifiers at display time, so new
translations do not require rewriting audit history.

## Authorization and tenant isolation

- `core.audit.append` writes tenant events.
- `core.audit.read.own` reads only `visibility=user` events related to the
  effective user in the current tenant.
- `core.audit.read.tenant` reads `user` and `tenant_admin` visibility in the
  current tenant.
- `platform.audit.read` reads all tenants, platform events, and all visibility
  levels.
- `platform.audit.retention.manage` authorizes retention administration.
- `platform.superadmin` inherits all of these through the standard privilege
  evaluator.

All restrictions are included in Core SQL predicates for lists, detail, and
filter options. Supplying another `user_id` or `tenant_id` cannot broaden the
caller scope. Tenant lists are returned only to platform readers.

## API

- `POST /api/v1/audit/events` appends a structured event.
- `POST /api/v1/audit/record` remains as a compatibility endpoint.
- `GET /api/v1/audit/events` lists authorized events.
- `GET /api/v1/audit/events/:id` returns an authorized event detail.
- `GET /api/v1/audit/filter-options` returns only values visible to the caller.

Multi-value filters use comma-separated values, for example
`application_id=a1,a2&severity=warning,error`. Supported filters are time range,
tenant, user, application, event type, category, severity, outcome, scope,
resource, and correlation ID. Results are ordered by `occurred_at DESC, id DESC`.
The opaque cursor contains both fields, preventing duplicates or omissions when
events share a timestamp. Responses use `{ items, next_cursor }`.

The Web Audit log page stores relative or absolute time ranges and active
filters in its query string. A compact filter builder uses searchable,
keyboard-accessible multi-select chips instead of native multi-select fields.
Application and tenant display names are primary while technical IDs remain
secondary. The responsive event table opens a structured Summary, Actor,
Target, Request, and Metadata drawer; raw JSON is an optional technical view.
Tenant selection remains hidden outside platform scope.

## Application writes

Core-managed runtime JWTs carry `core.audit.append`. Core derives tenant and
application identity from the token and rejects application attempts to write a
different tenant, platform scope, or `platform_admin` visibility. Request ID,
bounded `x-correlation-id`, IP, and user agent are captured consistently.

Inventory demonstrates the integration using only `HC_CORE_APP_TOKEN_FILE`. It
records item create/update/delete events. Core records authorization denials
that occur before the request reaches Inventory. Audit API failure does not roll
back an Inventory business operation; its logger receives a bounded technical
error without the token or full metadata payload.

## Sanitization and retention

Metadata recursively redacts password, secret, token, authorization, cookie,
API/private key, and client-secret variants. Depth, array length, string length,
user-agent length, and total serialized metadata size are bounded. Complete
request and response bodies are never captured automatically.

Retention defaults to 365 days with batches of 1,000 rows. Configure
`AUDIT_RETENTION_DAYS` and `AUDIT_RETENTION_BATCH_SIZE`, then explicitly run:

```shell
npm run audit:retention
```

Run it from a built Core workspace or with `docker compose exec core npm run
audit:retention`. There is deliberately no per-process interval. The command deletes only expired
rows in bounded batches and appends a platform audit event with the deleted
count. Operators schedule this command in their single maintenance runner.
