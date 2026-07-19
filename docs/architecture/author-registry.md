# Author Registry architecture

Status: Approved architecture; production deployment is pending.

The Author Registry is the official Talpaversum trust authority for author identities, public keys, certificates and revocations. It runs only as an official Talpaversum service. A technically compatible Registry operated by another party is not automatically trusted.

Trust begins with an explicitly provisioned and pinned Registry root trust anchor. Discovering a Registry URL or receiving a compatible response does not establish trust.

The Registry owns:

- official `author_id` identities;
- author public keys and root-signed certificates;
- author, key and certificate status;
- revocation and trust snapshots;
- Registry audit records.

The Registry does not operate customer licensing workflows, issue customer licenses, approve application runtimes or publish catalog entries. Those responsibilities belong to the issuer, Core runtime administration and catalog review respectively.

Human management flows through Core delegation and separate platform RBAC. Machine identities used in production remain an open hardening item and must not be replaced by a globally shared administrator credential.

See [author workflows](./author-workflows.md), [production operations](../operations/author-registry.md) and [production PKI](../operations/production-pki.md).
