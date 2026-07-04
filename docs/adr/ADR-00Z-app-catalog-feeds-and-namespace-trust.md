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
  a compose file, service name, project name, and internal base URL, but the MVP
  backend only returns an approval plan until the runtime manager is implemented.

The catalog entry stores `deployment_json` separately from the manifest. The
manifest describes integration with Core; deployment metadata describes how this
instance may run or reach the app.

Core-managed compose must remain an operator/admin capability. It must not
silently allow published ports or host mounts from a remote feed; those require
explicit policy and approval.

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

## Namespace Trust

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
