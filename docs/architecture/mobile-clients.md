# Application-owned mobile clients

Status: Open architectural design. This is not an implementation plan.

A mobile client is an optional presentation channel for an installed application backend. It is not a Core UI and must not become an alternative administration interface without a separately approved security model.

The design must decide:

- whether applications can be web-only, web-and-mobile or mobile-only;
- authentication, Core-instance and tenant discovery;
- token issuance, refresh, revocation and secure device storage;
- manifest metadata and client/backend version compatibility;
- direct application access versus Core-managed ingress;
- push notification trust and delivery;
- deep-link ownership and validation;
- device registration, removal and compromise response;
- offline synchronization, conflict handling and data protection;
- application-store distribution and publisher verification;
- whether any administrative action is permitted from a mobile client.

Implementation work must wait for the authentication, trust, tenant and distribution boundaries to be approved.
