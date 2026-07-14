# Deployment

Status: early development.

This section is intentionally short. HC is not production-ready yet; deployment
docs should describe what can be run now and name unresolved decisions without
pretending they are solved.

## Current supported path

Use Docker Compose for local self-hosted testing:

- PostgreSQL
- `hekatoncheiros-core`
- `hekatoncheiros-web`
- HTTP by default, with optional HTTPS ingress termination
- local image builds

Start here:

- [Docker deployment](./docker.md)

## Reference deployment modes

- [Baremetal / VM](./baremetal.md)
- [Kubernetes](./kubernetes.md)

These have reference configurations and validation steps, but are not yet
packaged production installers or Helm releases.

## Application runtime target

- **Core-managed:** Core acquires a versioned application package from a stable
  HTTPS catalog, validates it, starts it, and owns its runtime lifecycle.
- **External:** the operator starts and manages the application independently;
  Core only installs its integration from the running application's URL.

The current local HTTP package server and build-on-install flow are development
tools, not the production distribution model.

## Shared assumptions

- One canonical HC entrypoint is preferred, for example `https://hc.example.com`.
- The web shell owns user-facing UI routing.
- Core owns `/api/v1/*`, app lifecycle, and runtime UI plugin URLs.
- App-specific hostnames, such as `inv.example.com`, are future work.
- Docker Compose can terminate HTTPS at the optional nginx ingress; certificate
  acquisition and renewal remain deployment responsibilities.
- Docker Hub or another public image registry is not used yet.

## Database placement

Each deployment mode should support:

- a deployment-managed PostgreSQL instance
- an externally provided PostgreSQL instance

Defaults for now:

- baremetal: operator provides DB credentials
- Docker: Compose starts PostgreSQL
- Kubernetes: in-cluster PostgreSQL sketch, external DB later

## Open issue: standalone apps

Standalone app integration is unresolved.

The unresolved question is whether and how an app operated outside the target HC
instance can safely use the target HC tenant/app database model.

This still needs a decision for:

- DB credentials
- migrations
- backup and restore boundaries
- network policy
- disable/uninstall lifecycle

Until this is resolved, deployment docs should treat standalone apps as future
work.
