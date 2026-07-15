# Licensing Issuer Administration

`hc-app-licensing` is an author-operated issuer. Its administration workflow is
separate from tenant licensing in Core: Core activates, imports, validates, and
selects licenses, while the issuer creates and revokes them.

## Issue license workflow

The planned Issuer Admin UI uses three steps:

1. **Recipient** - select or create a customer reference, enter the target
   tenant, choose an author-owned application, and provide the Core instance ID
   when the license is instance-bound.
2. **License terms** - select portable or instance-bound mode, validity dates,
   features, entitlement values, and structured limits.
3. **Review and issue** - review the effective claims, issue the signed license,
   and provide both the license JWS and an offline bundle. An online activation
   handoff may be offered when the recipient has a registered OAuth client.

Issued material must never expose the author's private signing key. Every issue
operation records the operator identity, recipient, effective terms, resulting
`jti`, and timestamp in an audit trail.

## Required backend foundation

The current service exposes bearer-protected `POST /v1/licenses/issue`, but it
does not yet expose a secure issuer-operator session, product/customer
administration, grant listing, revocation administration, or audit APIs. A
production UI must not call the issuing endpoint with a shared token embedded
in browser code.

Before the UI becomes operational, define and implement:

- issuer operator authentication, authorization, session lifecycle, and CSRF
  policy;
- author-owned application and customer records;
- create, list, inspect, renew, and revoke operations for license grants;
- structured validation for features, entitlement values, and limits;
- auditable offline bundle download and online activation handoff;
- an explicit deployment decision: standalone author service or installable HC
  application with a manifest and UI plugin.

Until those boundaries are decided, issuing remains an API-level development
capability. Core must not present it as a tenant administration action.
