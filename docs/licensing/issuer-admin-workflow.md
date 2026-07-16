# Licensing Issuer Administration

`hc-app-licensing` is an author-operated, optional Hekatoncheiros application.
It issues licenses; Core only activates, imports, validates, selects, and
enforces them.

## Administration model

The UI is loaded as a Core web-shell plugin and all requests pass through the
Core application proxy. Core supplies a signed delegated identity. The issuer
checks granular `licensing.*` privileges and records the user, tenant,
permission, operation, target, request metadata, result, and correlation ID.
The issuer has no local accounts, login page, passwords, or sessions.

The administration surface covers:

- products, editions, capabilities, and default policy;
- customers and registered Core instances;
- commercial grants and their validity, limits, and offline policy;
- pending, approved, rejected, completed, and failed activations;
- issued licenses, replacement, suspension, revocation, and bundle download;
- certificate/signing-key state and issuer audit history.

## Issue lifecycle

1. Create an active product and customer.
2. Register the customer's Core instance and its public identity.
3. Create and activate a commercial grant.
4. Core submits an online activation, or an operator imports Core's signed
   offline activation request.
5. An operator with `licensing.activations.approve` approves or rejects it.
6. An operator with `licensing.licenses.issue` issues the JWS license bundle.
7. Core verifies the registry root, author certificate, author key, claims,
   audience, expiry, and cached revocations before storing the license.

Commercial grants and issued licenses remain separate records. Renewing or
replacing a license produces new signed material and preserves history.

## Key custody

The author private signing key stays in the issuer deployment and is never
returned through an API or uploaded to the registry. The registry stores only
public author keys and issues the author certificate. Rotation retains old
public verification material for licenses issued before the rotation.
