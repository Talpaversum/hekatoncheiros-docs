# Author workflow architecture

Status: Approved architecture; production-hosted services remain subject to the roadmap.

Hekatoncheiros separates three independent decisions: an instance capability says whether a service exists, RBAC says whether a user may operate that service, and author scope says which `author_id` the operation may access. Passing one check never implies passing either of the others.

## Official operating modes

There are exactly two official author operating modes:

- `talpaversum_hosted`: Talpaversum is the target operator for the reviewed build, runtime and managed issuer services.
- `trusted_self_hosted`: the approved author operates its runtime and single-author issuer while chaining identity to the official Registry certificate.

Private application development is not an official author operating mode. A private developer does not receive an `author_id`, is not registered in the official Author Registry and cannot publish directly to the official Talpaversum catalog.

Official authorship begins with an author request. Approval creates an official author identity and scoped owner membership. It does not approve an application, runtime, issuer or catalog submission; each remains a separate review and audit event.

## Instance capabilities

Core exposes authenticated instance capabilities and uses them together with configuration readiness. Existing private installations can use private development, trusted origins and private catalogs without enabling official services. Registry-dependent features remain unavailable until their capability and Registry endpoint are configured.

Capability gating is not authorization. Direct API calls must enforce the same capability, RBAC and author-scope boundaries as the Web interface.

## Author scope

Author-owned connections, applications, submissions, memberships and workflow events carry an `author_id`. Core derives access from active author membership and its permission set. A user with several memberships selects an active author, but Core validates that selection on every scoped request.

Repository credentials remain in Core, are encrypted at rest and are never returned by APIs or forwarded to Registry and licensing services. Source access never grants runtime, catalog, licensing or Registry privileges.

## Trust and outages

The official Author Registry is not a required dependency for private development. Private manifests and configured feeds remain local workflows when official services are absent.

Existing trust snapshots may be used according to the documented offline policy during a Registry outage. Operations that require current trust must fail explicitly. Author suspension or revocation blocks new trusted publication but does not silently remove installations, licenses or customer data.

## Related documents

- [Author and developer UI boundaries](./author-ui-boundaries.md)
- [Author Registry architecture](./author-registry.md)
- [Licensing architecture](./licensing.md)
- [Official author workflow guide](../guides/app-developer/author-workflows.md)
- [Private application development](../guides/app-developer/private-app-development.md)
