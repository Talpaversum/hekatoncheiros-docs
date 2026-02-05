# ADR-00X: Application Installation Registry

## Status
Accepted

## Context

Hekatoncheiros is an extensible platform that supports applications developed in separate repositories, potentially by third parties.
Applications are not part of the core codebase and must be treated as external components.

An earlier approach relied on a static `app-registry.json` file located outside the core repository, including relative paths to application repositories.
This created tight coupling between core and apps, violated separation of concerns, and prevented safe runtime installation.

## Decision

1. Application registry data is runtime state, not source code.
2. Core must not reference specific applications or their repository structure.
3. Core accesses installed applications exclusively through an abstract `AppInstallationStore`.
4. Application manifests are collected, validated, and normalized during installation.
5. Application installation is a separate process (e.g. marketplace or installer wizard).
6. All core subsystems (proxy, navigation, privileges) use the same installation store as the single source of truth.

## Consequences

- Clean separation between core and applications
- Support for third-party apps
- Runtime install and uninstall without code changes
- No filesystem or repository coupling

## Rejected Alternatives

- Static app registry files committed to a repository
- Relative filesystem references to application repositories
