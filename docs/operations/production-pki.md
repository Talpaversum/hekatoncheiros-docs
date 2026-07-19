# Production PKI operations

Status: Production target with unresolved key-storage decision.

Production root signing material is provisioned through an explicit ceremony outside normal Registry, Core or issuer startup. Production startup must never generate or replace a Registry root.

The development bootstrap creates disposable, environment-marked material only. A development root is not a production trust anchor and must never be promoted.

## Approved constraints

- Private root, author and issuer keys are absent from source repositories, images and Compose files.
- Public trust material is distributed separately from private signing material.
- Services fail closed when required production identity material is absent, malformed or marked as development/test material.
- Rotation preserves explicit key identifiers and a documented overlap or revocation policy.
- Backup and recovery must be tested without silently generating a replacement identity.
- Certificate issuance, rotation and revocation must be auditable.

## Open production work

The final storage and signing mechanism may use protected files, a secret store or KMS/HSM-backed operations. That choice, authorized roles, recovery, BYOK and compromise procedures remain the [production signing-key management decision](../architecture/signing-key-management.md).

This document does not create production keys. The implemented disposable workflow is documented in [PKI bootstrap](../licensing/pki-bootstrap.md).
