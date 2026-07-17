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

Registry and issuer configuration uses explicit identity modes:

| Mode | Purpose | Accepted key identifiers |
| --- | --- | --- |
| `production` | Real ecosystem trust | Must not be marked `dev-*` or `test-*` |
| `development` | Disposable local work | Must be marked `dev-*` |
| `test` | Automated tests and fixtures | Must be marked `test-*` |

The Registry publishes its mode as `environment` and the boolean
`production_trust` in `GET /v1/trust-anchor`. Mode labels are safety metadata;
trust still comes only from an explicitly pinned public root key and Registry
identity.

## Local development bootstrap

Run this command explicitly from `hc-author-registry`:

```bash
npm run pki:bootstrap:dev -- --author-id=talpaversum
```

It creates `.local/dev-pki/` containing:

- a disposable development Registry root key pair;
- an author signing key pair;
- a root-signed, 30-day development author certificate;
- public JWKS files;
- `registry.env` and `issuer.env` configuration snippets;
- a warning describing the material as development-only.

The directory is excluded from Git. Private JWK files and environment snippets
use mode `0600`; public material uses `0644`. The bootstrap refuses to overwrite
an existing output directory. Delete or archive the entire directory and choose
a new output path when a fresh disposable chain is needed.

The generated public Registry JWKS can be supplied to a local Core through
`LICENSING_ROOT_JWKS_JSON`. This only establishes trust in that disposable local
chain. It does not establish production trust.

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
