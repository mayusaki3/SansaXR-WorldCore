# World Access and Instance PoC — Request

## Goal
Implement a Rust-only reference for hybrid seamless-world access: physical boundaries plus isolated instances. SansaSphere and SansaVRM are external interfaces represented with fakes. Do not require an engine or live external services.

## Decisions
- Use SansaXR-WorldCore develop and HLDocS 0.6.1-preview.
- Access operations are separated: Discover, View, Enter, Interact, AssetFetch.
- AccessRegion is a spatial policy subject, not a streaming Cell. WorldDefinition is not WorldInstance.
- Physical boundary permission is enforced server-side. Failure leaves user in an authorized position.
- Separate-instance portal transition validates destination permission and uses an idempotent transition token. Denied, timed-out or disconnected transitions must not produce partial migration.
- Sandbox/test usage is segregated from Live usage.

## Proposed P0 contracts
- `SubjectContext`: principal ID, validated grants, validity time.
- `AccessRequest`: principal, world, instance, region, operation, context.
- `AccessDecision`: Permit / Deny / Indeterminate and reason code. Indeterminate must not grant protected access.
- `RegionBoundary`: geometry reference + protected operation set.
- `PortalDestination`: target WorldDefinition, instance selector, destination spawn.
- `Transition`: request ID, source session, target instance, state, lease.
- `UsageEvent`: event ID, environment (sandbox/live), world revision, content ID, coarse usage type.

## Test outline
- AUTH-001: public view but paid enter denied without entitlement.
- AUTH-002: invited subject enters private area; unrelated subject denied.
- AUTH-003: absence/expiry of authorization fails closed for paid entry.
- GEO-001: cross-cell region boundary is enforced exactly once.
- GEO-002: denied crossing preserves last authorized position.
- INST-001: authorized instance migration succeeds only once.
- INST-002: duplicate requests do not create duplicate sessions.
- INST-003: authorization denied or preload failure leaves original session valid.
- INST-004: disconnect around commit can resume idempotently.
- USAGE-001: sandbox events cannot count as live usage.
- USAGE-002: retransmission does not double count an event.

## Out of scope until cross-repo agreement
SansaSphere token specification, entitlement issuance/payment, revenue calculation, SansaVRM object schema, engine rendering/physics, iOS/Android adapter, production geometric precision.

## Dependency and delivery
Build testable Rust models with mock authorizer and in-memory repository first; document interfaces before integrating external services. The Worklog at ../Worklog/2026-10-23_initial_design_and_access.md holds larger context.
