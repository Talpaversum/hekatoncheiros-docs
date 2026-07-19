# Core2Core architecture

Status: Deferred. This architecture is not part of the current implementation scope.

Core2Core would allow independently operated Core instances to exchange selected application and catalog information without sharing a database or treating either instance as an automatic trust authority.

Deferred design areas include:

- remote application discovery and invocation;
- scheduled cross-Core catalog synchronization;
- application suggestion, review and publication workflows;
- protected namespace ownership across instances;
- OAuth or another explicit inter-instance trust establishment;
- separate database and tenant boundaries;
- signed, scoped cross-Core tokens;
- signed migration or deployment bundles;
- replay protection, revocation and outage behavior;
- audit attribution across instance boundaries.

No current API, catalog feed or private trusted origin should be described as Core2Core implementation.
