# Author and developer UI boundaries

Status: Approved architecture.

## Private App Development

Private App Development is available on self-hosted instances with the `privateAppDevelopment` capability.

It manages local application projects, trusted origins, manifests, private feeds, installation and local runtime verification.

It does not depend on Author Registry data.

A private developer has no official `author_id` and does not gain access to Author Workspace through this surface.

## Author Workspace

Author Workspace is available only to users with an active membership in an official author profile.

Every request is scoped to one explicit `author_id`.

Author Workspace manages:

- the selected author profile,
- team membership,
- Git connections,
- author applications,
- author licensing,
- catalog submissions,
- author activity.

## Author Administration

Author Administration is a platform-operator area.

It manages:

- author requests,
- request review,
- approval,
- rejection,
- suspension,
- revocation,
- author memberships.

It does not manage Registry root keys or application runtime approval.

## Registry Administration

Registry Administration manages only the cryptographic trust authority:

- author public keys,
- author certificates,
- author and key revocations,
- Registry trust anchor,
- trust snapshots,
- Registry health,
- Registry audit.

It does not manage application drafts, builds, runtime approval or catalog publication.

## Catalog Review

Catalog Review manages application submissions to the official catalog.

It does not turn a local catalog entry into an official submission automatically. Official catalog materialization is not currently complete.

## Runtime Review

Runtime Review manages hosted runtime deployment and execution approval.

It is distinct from source validation, catalog review and Registry trust administration. The hosted build worker remains planned work.
