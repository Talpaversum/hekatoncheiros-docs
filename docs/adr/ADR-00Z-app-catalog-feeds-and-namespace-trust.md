# ADR-00Z: Application Catalog Feeds and Namespace Trust

## Status

Accepted for MVP direction.

## Context

Hekatoncheiros instances need a way to discover applications that can be
installed into the local instance.

The catalog must support two use cases:

- an operator installs Core and develops an app locally
- an author publishes apps through a catalog feed so other instances can discover
  and install them

The catalog must not weaken author identity. A feed is only a distribution
channel; it must not become the authority that decides who owns an app namespace.

## Decision

1. Every instance may publish an application catalog feed.
2. Core may import catalog entries from local/manual sources or remote feeds.
3. Catalog entries describe available apps, not local runtime installation
   state.
4. Local installation state remains in Core's installation registry.
5. Runtime entitlement remains in Core's licensing model.
6. Author and namespace identity remains under the author identity authority.

## Installation And Deployment

Catalog install is an admin action. A license-required app may still be
installed; Core blocks tenant runtime use until a selected active license exists.

Core supports these install modes conceptually:

- `external`: install the app registry record and use the manifest's existing
  base URL.
- `stage_only`: store/return the selected deployment plan without creating the
  runtime installation entry.
- `compose`: Core-managed compose deployment. The catalog contract may describe
  a package URL and checksum, compose file, service name, project name, and
  internal base URL. The MVP runtime and explicit administrator approval are
  implemented; lifecycle hardening remains follow-up work.

The catalog entry stores `deployment_json` separately from the manifest. The
manifest describes integration with Core; deployment metadata describes how this
instance may run or reach the app.

Core-managed compose must remain an operator/admin capability. It must not
silently allow published ports or host mounts from a remote feed; those require
explicit policy and approval.

Starting a Core-managed runtime requires an explicit approval request bound to
the deployment plan, manifest SHA-256, and package SHA-256 shown to the
administrator. Core rejects missing or stale approvals before starting the
service and records
`platform.apps.runtime.start.approved` with the actor and deployment plan in the
audit log. If the audit write fails, runtime startup does not proceed.

## Publishing To This Instance Feed

Local installation does not automatically publish an app to other instances.
Core tracks feed publication separately from installation:

- `draft`: known locally but not published
- `pending`: proposed for publication, waiting for admin approval
- `published`: included in this instance's public feed
- `rejected`: reviewed and rejected for feed publication

The MVP supports direct admin publication. Admins may publish or unpublish
installed/enabled catalog entries from the catalog UI. The public feed endpoint
only emits entries with `published=true` and `publish_status=published`.

Future approval automation may use admin-issued publish tokens. A token can
represent pre-approval for a namespace, app, CI pipeline, author identity, or
limited time window. Submissions using such a token may move directly to
`published`; submissions without one should create a `pending` request for admin
review.

## Feed Model

The canonical feed endpoint is:

```text
/.well-known/hc/app-catalog.json
```

Feed items describe app releases:

```json
{
  "catalog_version": 1,
  "publisher": {
    "instance_id": "hcpi_...",
    "name": "Example HC Instance"
  },
  "items": [
    {
      "app_id": "talpaversum/inventory",
      "name": "Inventory",
      "version": "0.1.0",
      "manifest_url": "https://apps.example/.well-known/hc-app-manifest.json",
      "manifest_sha256": "...",
      "update_signal_jws": "eyJ...",
      "author_cert_jws": "eyJ...",
      "author_id": "aut_...",
      "author_namespace": "talpaversum",
      "license_required": true,
      "license_issuer_url": "https://licensing.example",
      "deployment": {
        "type": "compose",
        "compose_file": "docker-compose.app.yml",
        "service_name": "inventory",
        "internal_base_url": "http://inventory:3000"
      }
    }
  ]
}
```

MVP Core may store feed-ready metadata before automated feed sync is implemented.

The same endpoint is used for exporting this instance feed:

```text
GET /.well-known/hc/app-catalog.json
```

Import and export are intentionally separate. A Core instance can import many
feeds, but only entries explicitly published by its admin are exported.

MVP feed sync is operator-triggered:

1. Admin creates a feed source with `name`, `feed_url`, and `trust_mode`.
2. Admin runs sync manually.
3. Core fetches the feed JSON, then fetches each item manifest through the
   normal manifest fetcher.
4. If `manifest_sha256`, `app_id`, or `version` is present in the feed item,
   Core verifies it against the fetched manifest before upserting the entry.
5. Imported entries use `source_type=feed`; `manual` feed trust maps to
   `trust_status=unverified` until author/release verification exists.

An item may carry `update_signal_jws` together with `author_cert_jws`. Core
verifies the author certificate against its configured registry root JWKS and
then verifies the update signal with the author keys embedded in that
certificate. The `hc-app-update` signal must bind `sub` (the author-scoped
`app_id`), `app_version`, `manifest_sha256`, and `manifest_url` to the fetched
manifest, and its validity may not exceed seven days. If the application is
installed, the verified proof is persisted as a feed update signal. Supplying
only one of the two JWS values, an expired proof, or mismatched claims rejects
that feed item.

For local development, HTTP feed URLs are allowed only when their origin is in
Core's trusted origins list.

Automatic synchronization is disabled by default. It requires both
`APP_CATALOG_AUTO_REFRESH_ENABLED=true` on Core and
`auto_refresh_enabled=true` on the individual catalog source. Per-source opt-in
is accepted only for `verified` and `official` trust modes. The scheduler runs
at `APP_CATALOG_AUTO_REFRESH_INTERVAL_SECONDS` (minimum 60 seconds), prevents
overlapping runs, and isolates a failed source so other eligible feeds can
still refresh.

Every automatic source run records a platform audit event for success or
failure, including the source trust mode, import result, and allowed effects.
Changing a source's automatic-refresh opt-in is audited separately with the
administrator identity. The enforced automatic effect policy is
`catalog_metadata + update_signals` only. It explicitly rejects installed UI
artifact replacement and runtime mutation; those remain administrator-driven
operations with their existing approval and audit requirements.

## Namespace Trust

Core integrates with the private `hc-author-registry` for administrator-driven
author onboarding. With `AUTHOR_REGISTRY_URL` and
`AUTHOR_REGISTRY_ADMIN_TOKEN` configured, `platform.authors.manage` can create
an author, register public-only JWKS, and issue the first root-signed
`author_cert_jws` in one Core workflow. Key updates issue a replacement
certificate, and Core can snapshot the registry's public root JWKS and
revocation publication. Core stores the onboarding result and audit trail but
never accepts or stores author private key parameters.

Registry administration remains an explicit online operation. License and
signed-update verification continue to use pinned or synchronized public trust
material and must not require live registry access for normal application use.

`app_id` uses namespace format:

```text
<namespace>/<slug>
```

The namespace is not trusted just because it appears in a feed.

For verified distribution, Core must be able to validate:

- namespace owner
- author certificate
- release or manifest signature
- manifest hash
- revocation state

Protected namespaces may have stricter policy. For example:

```text
namespace: talpaversum
policy: official_feed_only
official_feed_url: https://catalog.talpaversum.example/.well-known/hc/app-catalog.json
```

This means third-party feeds cannot distribute entries that claim to be official
Talpaversum apps, even if the JSON shape is otherwise valid.

## Trust Modes

Core supports these conceptual trust modes:

- `dev`: local development, explicitly unsafe outside development
- `manual`: admin-approved manifest/feed without global verification
- `verified`: namespace and release verified through author identity
- `official`: verified and sourced from the namespace owner's official feed

## Role Of Author Registry

The author registry is not the application catalog.

It may own:

- author identities
- author keys and certificates
- namespace ownership
- root JWKS
- revocation snapshots
- optional official feed metadata for protected namespaces

It must not own:

- the list of all available apps
- local installed app state
- tenant licenses
- app artifacts
- pricing/catalog presentation metadata

## Open Follow-Up

The current `hc-author-registry` repository is intentionally kept separate for
now, but its long-term deployment boundary must be reviewed. It may become an
optional authority mode inside Core rather than a permanently separate service.
