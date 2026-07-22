# Licensing PKI Bootstrap

Status: **development and test bootstrap implemented; production ceremony deferred**

The licensing trust chain is:

```text
Registry root key -> author certificate -> license JWS
```

The Registry root is the highest authority in this chain. A production root
must not be created as part of application startup, a container build, a test
run, or the normal developer workflow.

## Trust environments

Registry and issuer configuration supports explicit identity modes:

| Mode | Purpose | Accepted key identifiers |
| --- | --- | --- |
| `production` | Real ecosystem trust | Must not be marked `dev-*` or `test-*` |
| `development` | Disposable local work | Must be marked `dev-*` |
| `test` | Automated tests and fixtures | Must be marked `test-*` |

The Registry publishes its mode as `environment` and the boolean
`production_trust` in `GET /v1/trust-anchor`. Mode labels are safety metadata;
trust still comes only from an explicitly pinned public root key and Registry
identity.

## Local Registry bootstrap

Run this command explicitly from `hc-author-registry`:

```bash
npm run pki:bootstrap:local
```

It creates an ignored `.env` and Registry root pair under `.local/pki`. The
private file and `.env` use mode `0600`; the public JWKS uses `0644`. Existing
keys and operator configuration are preserved. Explicit `--force` rotation
changes the trust anchor and invalidates material issued under the previous
local root.

The generated public Registry JWKS can be supplied to a local Core through
`LICENSING_ROOT_JWKS_JSON`. Registry runs with `REGISTRY_ROOT_MODE=production`
to exercise production file loading and validation, but the generated
`local-registry-root-*` key establishes only disposable local trust. It is not
an approved production root.

The bootstrap is Registry-only. Author signing keys, author certificates, and
Licensing issuer configuration are provisioned through the separate author
identity workflow; they are not emitted into shared environment snippets.

## Runtime validation

`hc-author-registry` fails before listening when root configuration is missing,
contains placeholders, exposes private material in public JWKS, contains an
invalid Ed25519 key, mismatches private and public keys, or conflicts with the
selected root mode. It never generates a root during startup.

`hc-app-licensing` fails before listening unless all of these checks pass:

1. Registry root JWKS is non-empty, public-only Ed25519 material.
2. `author_cert_jws` is signed by that root and the expected Registry issuer.
3. Certificate `registry_id` and subject match configured identities.
4. The certificate embeds a non-empty public author JWKS.
5. The issuer private key is valid and its public half is present in that JWKS.
6. Identity material is compatible with the selected production, development,
   or test mode.

Before an issued license is stored, the issuer verifies its signature against
the active author certificate and checks the license issuer identity. Core
independently repeats the root, certificate, license signature, namespace,
tenant, audience, expiry, and revocation checks during import or activation.

## Production root ceremony

No production Registry root is created by the current bootstrap. Its future
operational procedure must be approved separately and define at least:

- generation environment and authorized participants;
- private-key custody, online/offline signing boundary, and access control;
- encrypted backups, recovery testing, and disaster handling;
- root and author-key rotation with overlap periods;
- compromise detection, emergency revocation, and trust-store distribution;
- auditable certificate issuance and operator separation of duties.

Until that procedure is approved and real trust material is injected through a
secret-management boundary, `production` deployments must fail closed.
