# Author workflows

Hekatoncheiros separates application authorship from platform administration.
An author membership grants access only to one `author_id`; it never grants
platform-wide privileges.

## Operating modes

| Mode | Registry | Runtime and build | Licensing | Official catalog |
| --- | --- | --- | --- | --- |
| Talpaversum-hosted | Required | Talpaversum-operated; first start and dangerous changes require operator approval | Talpaversum-managed, isolated per author | Allowed after review |
| Trusted self-hosted | Required | Author-operated | Author-operated issuer, chained to the Registry certificate | Allowed after external identity validation and review |
| Private self-hosted | Not used | Local administrator-operated | Optional local/private issuer | Not allowed |

The official Author Registry exists only on the Talpaversum instance. A
private installation must remain usable when that Registry is absent.

## Official author onboarding

1. Open **Author Portal > Become an Author** and save a draft.
2. Select hosted or trusted self-hosted mode, complete the request, accept the
   terms, and submit it.
3. A platform operator starts review and approves, rejects, or requests
   changes. Review, suspension, and revocation actions are audited.
4. Approval creates an `author_id`, owner membership, scoped permissions, and
   an Author Registry identity and certificate.

For hosted mode, Talpaversum creates and encrypts the author signing key. For
trusted self-hosted mode, the author supplies public JWKS and retains the
private key. Private keys must never be pasted into Core or committed.

## Git and application workflow

GitHub is the first provider implementation. Use a fine-grained token with
read-only contents access to the smallest possible repository set. Core
validates the token against GitHub, encrypts it with
`AUTHOR_GIT_TOKEN_ENCRYPTION_KEY`, never returns it through the API, and
removes the stored credential when disconnected.

1. Connect GitHub to an approved author profile.
2. Select an accessible public or private repository, branch, and manifest
   path.
3. Validate the manifest. The `app_id` namespace must match the approved
   `author_id`.
4. Create and submit the application draft.
5. For hosted mode, request runtime approval. Operators must review deployment
   source, ports, volumes, environment, Docker socket use, network access,
   health checks, origins, and requested privileges.
6. Submit an eligible hosted or trusted external app to the official catalog.
   Publication and unpublication require catalog-operator review.

Repository tokens provide source access only. They do not grant runtime,
catalog, licensing, or Registry privileges.

## Private self-hosted development

Private development is not an official author request:

1. Run Core and Web.
2. Build and run the application backend on local infrastructure.
3. Add its exact scheme, host, and port under **Platform configuration >
   Trusted origins**. Review the HTTP warning and test connectivity.
4. Expose a valid manifest or private catalog feed from that origin.
5. Add the feed or install the application URL in **Applications**.
6. Use no license, or configure a local/private issuer according to local
   policy.

Core shows this application as private/unverified. Trusting an origin permits
Core to fetch application material from it; it does not establish official
author identity.

## Permissions and lifecycle

Author roles are owner, manager, developer, licensing manager, and viewer.
Their `author.*` permissions are checked against the selected `author_id`.
Platform operators retain separate `platform.authors.manage`,
`platform.catalog.manage`, `platform.apps.runtime.manage`, and
`platform.author_registry.manage` privileges.

Suspending or revoking an official author updates Core and the Registry.
Consumers must refresh Registry revocations. Catalog and runtime disablement
remain explicit operator actions so existing installations can be handled
without silently deleting customer data.

## Development trust

`hc-author-registry/scripts/bootstrap-dev-pki.mjs` creates a disposable local
root, author key, certificate, and environment snippets under ignored
`.local/dev-pki`. This root is marked `development`, is not production trust,
and must never be promoted. Production Registry root creation is a separate
offline operational ceremony with an approved lifecycle, custody, backup,
rotation, revocation, and recovery procedure.

EN and CS portal translations are complete. Until native portal translations
are supplied for SK, DE, FR, and ES, those locales deliberately use the
documented per-key English fallback.
