# Production signing-key management

Status: Open decision.

Production signing material must be provisioned outside normal application startup. Development bootstrap keys are disposable and cannot be promoted.

The decision must cover:

- encrypted files versus a secret store versus KMS/HSM-backed signing;
- Registry root-key custody and offline/online boundaries;
- hosted and trusted self-hosted author keys;
- issuer signing keys and per-author isolation;
- encrypted backup and tested restoration;
- disaster recovery and loss of signing access;
- scheduled and emergency rotation;
- key, certificate and root revocation;
- compromise detection and operator separation of duties;
- bring-your-own-key (BYOK) policy for hosted authors.

No production root or private key is created by this document. See [production PKI operations](../operations/production-pki.md).
