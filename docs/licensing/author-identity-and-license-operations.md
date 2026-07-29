# Author identity and license operations

This guide describes the operational contract implemented by Author Workspace,
Core, the official Author Registry, and hosted or self-hosted issuers.

Private application development does not create an official author identity. Author identity exists only after approval by the official Author Registry.

## Identity, keys, and certificates

The Registry assigns the stable `author_id`. An author certificate binds that ID
to public signing keys and is signed by the Registry root. A JWKS is key material,
not an identity by itself. Verification follows Registry root → author certificate
→ signed application update or license.

Talpaversum-hosted authors use non-exportable, encrypted managed private keys.
Core provisions their issuer, rotates keys through audited requests, and signs
license artifacts. Trusted self-hosted authors retain their private keys, upload
only public JWKS, configure an HTTPS issuer, pass operator verification, and sign
artifacts locally. Private keys and complete license secrets must never enter logs,
audit metadata, or public configuration bundles.

## Products, types, requests, and ownership

A licensing product connects commercial licensing configuration to one verified
application. A license type defines its allowed owner scopes, duration, request
and approval policy, capabilities, limits, activation rules, offline support, and
future pricing metadata.

A license requester and a license owner are separate identities. The requester is the user who initiated the workflow. The owner is the instance, tenant or user to which the resulting license applies.

User ownership applies only to that user. Tenant ownership applies to all eligible
users in the tenant. Instance ownership applies across the Core instance. Runtime
entitlement evaluation accepts a valid user, tenant, or instance license; a
narrower license cannot remove capabilities granted by a broader valid license.

Requests may require tenant or platform-local approval before author review.
Author review can request information, approve, reject, or record a future-payment
state. Payment metadata is only a workflow boundary: this implementation does not
charge, invoice, or imply that approval means payment.

## Grants, signing, and direct assignment

Approval creates a grant: the durable authorization describing owner,
capabilities, limits, and validity. Issuance creates a separate signed license
artifact. Direct assignment creates an approved grant without a requester, then
uses the same issuance path.

Hosted issuance signs claims containing `author_id`, application, product, license
type, grant, explicit owner scope and owner ID, capabilities, limits, validity,
serial number, and JTI. Self-hosted issuance must return an artifact whose
Registry certificate, author, application, grant, and owner all match. The
application publisher and license `author_id` must be identical.

## Activation and delivery

Issuance and activation are separate audited operations. Supported delivery paths
are online activation, offline request/response, manual bundle download, and
managed automatic activation. Pending activations can be approved or rejected;
failed or rejected operations can be retried; active operations can be revoked.
Offline responses are downloadable only after approval.

Tenant-owned artifacts are imported into the target tenant. User- and
instance-owned artifacts remain scoped to their respective owners and are
evaluated only in matching request context.

## Renewal, replacement, suspension, and revocation

Renewal issues a replacement artifact and marks its predecessor as replaced.
Suspension is reversible operational blocking. Revocation is permanent for the
artifact: its JTI is added to local revocations and active activations are revoked.
The UI must preview the affected application, owner, and active activation count
before a destructive lifecycle decision.

Notifications cover request submission and local approval, information requests,
approval/rejection, issuance, activation, expiry, suspension/revocation,
certificate expiry, key rotation, and issuer verification failures. Initial
delivery uses platform notification records; email delivery may be added without
changing the lifecycle contract.

## Authorization and audit

Author permissions are scoped to exactly one `author_id`. Recommended roles are
Author Owner, Identity Administrator, Licensing Manager, License Reviewer,
Support Operator, and Auditor. None grants Registry-root administration.

Every identity and licensing operation records its actor, author, affected
objects, ownership scope, state transition, reason, and correlation ID. Registry
operators alone approve author identities, issue or revoke author certificates,
manage root trust, and view global Registry audit.

