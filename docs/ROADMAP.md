# Hekatoncheiros Roadmap

This roadmap contains only current and future work.

Completed platform capabilities are documented in [`STATUS.md`](../STATUS.md). Detailed implementation history belongs in [`CHANGELOG.md`](../CHANGELOG.md). Architecture and operational rules belong in the relevant documents under [`architecture/`](./architecture/) and [`operations/`](./operations/).

Task status:

* `[ ]` planned or in progress,
* completed work is removed from this roadmap after being recorded in `STATUS.md` and `CHANGELOG.md`,
* unresolved design questions are listed under **Open decisions**.

---

## Current objective

Complete a production-ready application lifecycle covering:

1. application development,
2. source integration and builds,
3. author verification,
4. catalog publication,
5. licensing,
6. installation,
7. runtime operation,
8. updates and rollback,
9. health monitoring and diagnostics.

---

# Now

## Hosted application delivery

**Outcome:** A Talpaversum-hosted author can connect a repository and produce a reviewed, immutable application deployment without operating their own infrastructure.

**Status:** In progress

### Required work

* [ ] Connect approved hosted-author applications to a build worker.
* [ ] Check out the exact selected Git revision.
* [ ] Validate the application manifest before the build starts.
* [ ] Produce an immutable runtime package with recorded hashes.
* [ ] Generate a deployment plan compatible with the existing Core runtime manager.
* [ ] Store build artifacts in a controlled artifact store.
* [ ] Connect build, validation, runtime approval and deployment history.
* [ ] Complete update and rollback behavior for hosted applications.
* [ ] Ensure build credentials and repository secrets are not included in artifacts or logs.

### Done when

* An author can select a Git revision and request a build.
* The produced package is immutable and reproducible from recorded source metadata.
* Core verifies the package and deployment-plan hashes before runtime approval.
* A failed update does not replace the last working deployment.
* Build, deployment and rollback events are attributable and audited.

### Dependencies

* Developer Tools project lifecycle
* Git provider connections
* Runtime approval
* Artifact storage
* Application manifest validation

---

## Official catalog publication

**Outcome:** Approved hosted and trusted self-hosted applications can be published into the official Talpaversum catalog.

**Status:** Planned

### Required work

* [ ] Materialize approved catalog submissions into the public catalog feed.
* [ ] Publish immutable manifest and package references for hosted applications.
* [ ] Support trusted external manifest or feed references for self-hosted authors.
* [ ] Validate external issuer discovery and author certificate chains.
* [ ] Store the exact approved application version and source revision.
* [ ] Support publish, unpublish, suspend and revoke operations.
* [ ] Define update review rules for already-published applications.
* [ ] Expose catalog publication status in Author Workspace.
* [ ] Expose review and publication actions in Catalog Administration.

### Done when

* A hosted application can be published with immutable package references.
* A trusted self-hosted application can be published with a verified external issuer.
* Private applications cannot enter the official catalog.
* Publication of a new version cannot silently change previously approved runtime or trust metadata.
* Catalog entries clearly identify hosted and externally operated applications.

### Dependencies

* Author Registry
* Author Workspace
* Catalog review workflow
* Hosted application delivery
* External issuer validation

---

## Managed hosted licensing

**Outcome:** Talpaversum can provide licensing as a managed service to multiple hosted authors with strict isolation.

**Status:** Planned

### Required work

* [ ] Add a managed issuer namespace for each `author_id`.
* [ ] Scope products, customers, Core instances, grants, activations, licenses and audit events by `author_id`.
* [ ] Assign a separate signing identity to every hosted author.
* [ ] Extend Core delegation tokens with author identity and author-scoped permissions.
* [ ] Prevent cross-author reads, writes, activation approval and license issuance.
* [ ] Connect hosted author approval to signing-key and certificate provisioning.
* [ ] Implement author-specific certificate and signing-key lifecycle.
* [ ] Add cross-author isolation tests.
* [ ] Keep the existing single-author issuer mode for trusted self-hosted authors.

### Done when

* Two authors can use one managed issuer service without sharing data.
* Each author’s licenses are signed by that author’s own signing identity.
* An author cannot read, modify or issue licenses for another author.
* All hosted licensing operations identify both the human operator and `author_id`.

### Dependencies

* Production Author Registry
* Production PKI
* Author-scoped Core delegation
* Secure signing-key storage

---

## Production Author Registry and PKI

**Outcome:** The official Talpaversum trust infrastructure can be operated securely outside development mode.

**Status:** Planned

### Required work

* [ ] Define and approve the production root-key lifecycle.
* [ ] Provision the Registry root key outside normal application startup.
* [ ] Define secure backup, recovery, access and rotation procedures.
* [ ] Replace all development and Compose trust material.
* [ ] Deploy the production Author Registry with persistent storage.
* [ ] Connect Core to the production Registry.
* [ ] Provision Core delegation signing keys.
* [ ] Provision author signing keys and certificates.
* [ ] Provision issuer signing identities.
* [ ] Verify trust-anchor and revocation synchronization.
* [ ] Verify Registry recovery without generating a replacement root.
* [ ] Document production operational procedures.

### Done when

* Production startup never generates root signing material.
* The Registry uses an externally supplied production database.
* Core validates authors against the production trust anchor.
* Root, author and issuer private keys are absent from source repositories and Compose files.
* Backup and recovery procedures have been tested.

### Dependencies

* Root-key lifecycle decision
* Production secret storage
* Registry deployment documentation

---

## Instance feature settings

**Outcome:** Instance administrators can control optional platform features without rebuilding or manually editing the Web application.

**Status:** Planned

### Required work

* [ ] Add administrative settings for supported optional capabilities.
* [ ] Distinguish `supported`, `configured` and `enabled`.
* [ ] Allow administrators to enable or disable Developer Tools.
* [ ] Allow administrators to enable or disable custom feeds and private catalogs.
* [ ] Allow official instances to enable author onboarding and Registry administration.
* [ ] Allow hosted author services to be enabled independently.
* [ ] Enforce settings in Core routes, Web routes and navigation.
* [ ] Record feature-setting changes in the audit log.
* [ ] Add safe migration defaults for existing private installations.

### Done when

* Disabled features are unavailable in both Web and Core APIs.
* A private installation does not require Author Registry configuration.
* Enabling a feature does not imply that it is correctly configured.
* Web displays the reason when a supported feature is not configured or enabled.

### Dependencies

* Instance capability model
* Core Console configuration
* RBAC

---

# Next

## Production licensing lifecycle verification

**Outcome:** The full online and offline licensing lifecycle is validated against production-like services and persistent databases.

### Required work

* [ ] Exercise the complete online lifecycle:

  * author onboarding,
  * product creation,
  * customer creation,
  * Core instance registration,
  * grant approval,
  * license issuance,
  * activation,
  * renewal or replacement,
  * suspension,
  * revocation enforcement.
* [ ] Exercise the signed offline activation-request and license-response flow.
* [ ] Test stale, revoked, malformed and mismatched artifacts.
* [ ] Verify restart and recovery behavior with persistent issuer data.
* [ ] Verify author, key, certificate and license revocations separately.
* [ ] Confirm that previously issued offline licenses follow the documented revocation policy.

### Done when

* All lifecycle transitions pass end-to-end integration tests.
* Negative and malformed cases fail with stable error codes.
* Audit records identify the actor, author, tenant, application and affected license.

---

## Scheduled trust and revocation synchronization

**Outcome:** Core automatically maintains current Registry and issuer revocation information while retaining manual recovery actions.

### Required work

* [ ] Add scheduled Registry trust synchronization.
* [ ] Add scheduled author and key revocation synchronization.
* [ ] Add scheduled issuer-license revocation synchronization.
* [ ] Store the last successful synchronization time.
* [ ] Expose stale-trust warnings in Core Console.
* [ ] Retain manual refresh as an administrative recovery action.
* [ ] Define behavior for temporarily unavailable Registry and issuer services.
* [ ] Add retry, backoff and failure auditing.

### Done when

* Existing installations continue operating according to the documented offline policy during temporary outages.
* New trust-dependent operations fail safely when required data is unavailable.
* Administrators can see the age and status of cached trust data.

---

## Scoped service identities

**Outcome:** Temporary shared administration-token compatibility paths are removed from production machine communication.

### Required work

* [ ] Define scoped identities for Core-to-Registry communication.
* [ ] Define scoped identities for Core-to-Issuer communication.
* [ ] Define scoped identities for Registry-to-Issuer communication where required.
* [ ] Support credential rotation and revocation.
* [ ] Identify the calling instance or service in audit records.
* [ ] Remove production dependencies on shared `ADMIN_TOKEN` credentials.
* [ ] Keep any compatibility token explicitly limited to development or migration.

### Done when

* Compromise of one service credential does not grant global administrative access.
* Every machine operation is attributable to a specific service identity.
* Credentials can be rotated without rebuilding applications.

---

## Production deployment hardening

**Outcome:** Registry, issuer, Core and Web have documented and tested production deployment paths.

### Required work

* [ ] Define supported Compose production topology.
* [ ] Define persistent storage and backup requirements.
* [ ] Define ingress and TLS termination.
* [ ] Define secret injection and rotation.
* [ ] Define database migration and rollback procedures.
* [ ] Add readiness and liveness checks.
* [ ] Verify restart and upgrade behavior.
* [ ] Review reported npm dependency advisories.
* [ ] Upgrade affected packages without forced or behavior-breaking changes.
* [ ] Document remaining accepted risks.

### Done when

* Production services do not depend on development Compose databases or development keys.
* Upgrades and database migrations have tested recovery procedures.
* Known production dependency advisories are fixed or explicitly accepted with mitigation.

---

## Integration and failure-path testing

**Outcome:** Critical author, Registry, licensing, catalog and runtime boundaries are protected by integration tests.

### Required work

* [ ] Expand Registry lifecycle tests against a persistent test database.
* [ ] Expand issuer lifecycle tests around database-backed transitions.
* [ ] Test authorization denials and author-scope isolation.
* [ ] Test audit-write failure paths.
* [ ] Test Registry and issuer outages.
* [ ] Test package, manifest and deployment-plan mismatches.
* [ ] Test catalog publication and unpublication.
* [ ] Test hosted and trusted self-hosted workflows independently.

### Done when

* Tests validate actual behavior rather than only source-code structure.
* Cross-author and cross-tenant access attempts are denied.
* Failures do not leave partially approved, published or activated objects.

---

# Later

## Core clustering

**Outcome:** Multiple Core instances can cooperate as one deployment.

### Planned scope

* [ ] Define cluster membership.
* [ ] Define leader or coordinator election.
* [ ] Define master-worker responsibilities.
* [ ] Support load balancing across Core instances.
* [ ] Coordinate scheduled jobs.
* [ ] Coordinate runtime ownership.
* [ ] Define shared-state and failure-recovery behavior.
* [ ] Prevent duplicate deployment, migration and synchronization work.

Implementation must not begin before the clustering architecture is approved.

---

## Deployment packaging

**Outcome:** Validated deployment references become supported installation packages.

### Planned scope

* [ ] Package the bare-metal/VM reference deployment.
* [ ] Provide supported upgrade procedures.
* [ ] Decide whether to publish a Helm chart.
* [ ] Package Kubernetes manifests or operators.
* [ ] Define compatibility and support levels for each deployment type.

Current bare-metal/VM and Kubernetes configurations remain reference deployments until this work is completed.

---

# Open decisions

Open decisions are design questions. They must be resolved in architecture documents before implementation tasks are added to **Now** or **Next**.

## Standalone application database ownership

Status and design questions: [Application database ownership](./architecture/application-database-ownership.md).

---

## Production signing-key management

Status and design questions: [Production signing-key management](./architecture/signing-key-management.md).

---

## Application-owned mobile clients

Status and design questions: [Application-owned mobile clients](./architecture/mobile-clients.md).

---

# Deferred

## Core2Core integration

Core2Core remains outside the current implementation scope. See [Core2Core architecture](./architecture/core2core.md).

---

# Current deployment position

Current support levels and production requirements are maintained in [Deployment operations](./operations/deployment.md).
