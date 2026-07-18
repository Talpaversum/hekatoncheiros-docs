# Official author workflows

Hekatoncheiros distinguishes between private application development and official authorship.

Private application development is not an author operating mode. A private developer does not receive an `author_id`, is not registered in the official Author Registry and cannot publish applications to the official Talpaversum catalog.

## Official author operating modes

There are exactly two official author operating modes.

| Mode | Official Registry | Runtime and build | Licensing | Official catalog |
| --- | --- | --- | --- | --- |
| Talpaversum-hosted | Required | Operated by Talpaversum | Managed by Talpaversum and isolated by `author_id` | Allowed after application and catalog review |
| Trusted self-hosted | Required | Operated by the author | Operated by the author and chained to the Talpaversum author certificate | Allowed after external issuer validation and catalog review |

Private self-hosted application development is documented separately in `private-app-development.md`.

## Official author onboarding

1. Open **Author Portal > Become an Author** and save a draft.
2. Select hosted or trusted self-hosted mode, complete the request, accept the
   terms, and submit it.
3. A platform author reviewer starts review and approves, rejects, or requests
   changes. Author request review is audited independently of Registry,
   catalog, external issuer, application, and runtime decisions.
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

## Permissions and lifecycle

Author roles are owner, manager, developer, licensing manager, and viewer.
Their `author.*` permissions are checked against the selected `author_id`.
Platform operators use separate privileges:

- author requests: `platform.authors.review`,
- Registry read, key, certificate, revocation, and audit operations:
  the matching `platform.author_registry.*` privilege,
- catalog review: `platform.catalog.manage`,
- hosted runtime review: `platform.apps.runtime.manage`.

Each privilege is also gated by the corresponding instance capability. A
successful author request creates the identity and membership only. It does
not approve an application, external issuer, catalog submission, or runtime.

Registry key rotation and revocation are separate Registry operations.
Consumers must refresh Registry revocations. Catalog publication and runtime
disablement remain explicit operator actions so existing installations can be
handled without silently deleting customer data.

## Development trust

`hc-author-registry/scripts/bootstrap-dev-pki.mjs` creates a disposable local
root, author key, certificate, and environment snippets under ignored
`.local/dev-pki`. This root is marked `development`, is not production trust,
and must never be promoted. Production Registry root creation is a separate
offline operational ceremony with an approved lifecycle, custody, backup,
rotation, revocation, and recovery procedure.

Translation completeness is verified by the Web i18n parity test. Documentation must not claim that a locale is complete unless the current test confirms that all required keys and interpolation placeholders exist.
