# Author workflow acceptance matrix

This matrix is the release gate for private development, official author
workflows, author isolation, licensing isolation, and Registry deployment.
All commands run from the named repository.

Passing these implementation gates does not make the hosted build, official catalog, managed issuer or Registry production-ready. Production readiness additionally requires the deployment and operational validation listed in [platform status](../../STATUS.md).

## Automated suites

| Area | Automated coverage | Command |
| --- | --- | --- |
| Private instance wizard | `hekatoncheiros-core/tests/developer-project-workflow.test.ts`, capability and navigation checks in the Web production build | `npm test -- --run` in Core; `npm run build` in Web |
| Official hosted workflow | `author-workflow-policy.test.ts`, `author-secret-store.test.ts`, `app-runtime-approval.test.ts`, `administrative-boundaries.test.ts` | `npm test -- --run` in Core |
| Trusted self-hosted workflow | `author-workflow-policy.test.ts`, `license-signature-chain.test.ts`, external issuer boundary in `administrative-boundaries.test.ts` | `npm test -- --run` in Core |
| Author workspace isolation | `author-scope-isolation.test.ts` | `npm test -- --run` in Core |
| Hosted licensing isolation and signing identity | `hc-app-licensing/tests/author-isolation.test.ts`, `issuer-identity.test.ts`, `delegated-auth.test.ts` | `npm test -- --run` with explicit `TEST_DATABASE_URL` in Licensing |
| Registry delegation | `hc-author-registry/tests/delegated-auth.test.ts`, Core `author-registry-client.test.ts` | `npm test -- --run` in Registry and Core |
| Registry migrations and readiness prerequisites | `hc-author-registry/tests/migration-discovery.test.ts`, `root-key-config.test.ts` | `npm test -- --run` with explicit `TEST_DATABASE_URL` in Registry |
| Registry release contents | `npm run release:check` | `npm run release:check` in Registry |
| Contextual help and locales | `hekatoncheiros-web/scripts/check-localization.mjs` | `npm run test:localization` in Web |

## Required behavior

### Private instance

- `privateAppDevelopment`, `trustedOrigins`, and optional `privateCatalogs`
  drive Developer Tools without Registry configuration.
- The wizard tests the origin, records explicit trust, validates a manifest or
  private feed, installs the app, and marks it local/unverified.
- Official author and Registry navigation is absent when its capability is not
  available. Capability-filtered help never directs a private instance to the
  Author Registry.

### Talpaversum-hosted author

- Author request approval creates an author ID, owner membership, Registry
  certificate, and encrypted author signing key.
- Git source, application review, hosted runtime review, author-scoped hosted
  licensing, and catalog review remain distinct operations and audit events.
- Licensing requests carry the explicit author ID, and the licensing service
  obtains signing material for that author only through Core custody.

### Trusted self-hosted author

- The onboarding request supplies public JWKS and an external issuer URL.
- Author approval creates the Registry identity but leaves the external issuer
  in `pending_review`.
- Official catalog submission is blocked until a catalog operator separately
  approves the external issuer.
- Runtime and licensing stay externally operated.

### Isolation

- Every author workspace API URL contains `author_id` and membership checks
  happen before reads and writes.
- Switching the active author changes every author query key and URL.
- Licensing tables, delegated authorization, and signing identities are
  isolated by author ID, including two authors in the same tenant.

### Registry deployment

- Production Compose requires an external `DATABASE_URL` and has no PostgreSQL
  service or automatic dev fallback.
- Development PostgreSQL starts only through the explicit dev Compose command.
- `/health/live` is process liveness; `/health/ready` checks the database,
  migration table, complete known migration set, and root configuration.
- The migration integration test uses only `TEST_DATABASE_URL`, runs twice,
  and skips explicitly when it is absent.
- Release validation rejects `.env`, private JWK/key material, and real database
  credentials.
