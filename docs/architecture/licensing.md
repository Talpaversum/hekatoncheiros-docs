# Licensing architecture

Status: Approved boundaries; managed hosted operation is not production-ready.

## Core consumption and enforcement

Core consumes licenses and enforces navigation and API access for installed applications. It validates the Registry root, author certificate, license signature, namespace, tenant, audience, validity interval and known revocations. Core also owns tenant activation, offline import and the binding between an installed application and a selected entitlement.

Core is not the commercial issuer and does not hold an author's issuer private key.

## Trusted self-hosted issuer

The licensing application implements `single_author` operation for a trusted self-hosted author. The author operates the service and database and supplies an author certificate and matching issuer key chained to the official Registry trust anchor. Core delegates authenticated human management requests to the issuer.

## Talpaversum-hosted issuer

The issuer source contains a `managed_multi_author` foundation with `author_id` data scoping, delegated author context and isolation tests. This implementation foundation is not equivalent to an available managed service.

Production managed hosting still requires per-author signing identities, production provisioning, secure key storage, cross-author operational validation, deployment and recovery procedures. Until those are complete, documentation and UI must not promise managed hosted licensing as automatically available.

## Trust boundaries

The Author Registry certifies author public keys. It does not run customer, product, grant, activation or license workflows. An issuer signs licenses; Core independently verifies and enforces them. Each layer retains its own audit and persistence boundary.

See [trust model](../licensing/trust-model.md), [Core integration](../licensing/core-integration.md), [issuer workflow](../licensing/issuer-admin-workflow.md) and [issuer operations](../operations/licensing-issuer.md).
