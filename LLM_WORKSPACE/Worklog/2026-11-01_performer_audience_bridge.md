# Performer/Audience Instance Bridge — 2026-11-01

## Status
Design proposal for SansaXR-WorldCore develop. Not yet approved as canonical specification. HLDocS v0.6.1-preview Worklog.

## User requirement
- Performer side A consists of multiple candidate instances, but only one source is active at a time, analogous to broadcast relay switching.
- Audience side B comprises multiple independent instances with seats and local state.
- Connected mode: A and B behave as if colocated, and performers in A can see audience in B.
- Broadcast mode: B sees A as in connected mode, but A is not virtually present in B; A does not need direct B awareness.
- Stage gimmicks in B must be controllable by A's live system.

## Proposed architecture
- EventCoordinator: event clock, event ID, active A source ID and monotonically increasing source epoch.
- LiveBridge: relays selected A state/media and authenticated event/gimmick commands to B instances; supports Connected and Broadcast modes per event/zone.
- PerformerProjection: a non-authoritative representation of A in B, never a duplicate authoritative performer.
- AudienceProjection: optional, filtered/aggregated representation of B participants in A for Connected mode.
- AudienceInstance B: authoritative for seats, local movement, nearby interaction and local gimmick execution.
- GimmickEndpoint B: validates capability/target/epoch/event time/command ID, schedules and executes commands; reports ACK/status.
- Live media transport is a separate capability provider; WorldCore owns routing, synchronization, permissions and lifecycle, not video encoding/large-scale distribution.

## Semantics and limitations
- Connected means bidirectional perception/interactions via projections and messages, not literally merging all simulation state.
- With many B instances, rendering all audience avatars into A is unbounded; use selected zones, nearby viewers, summaries or capped representations. Selection policy requires user decision.
- Broadcast means A-to-B presentation only by default; optional audience reactions may be separately aggregated without making B appear as a shared space to A.
- Performer projections in B must not be treated as authoritative physical actors; B physics/gimmicks must be executed by B runtime.
- Exactly one active A source per event; reject stale source epoch and stale/duplicate gimmick commands.
- For time-critical effects, send scheduled commands with event timestamp; use ACK and reconciliation snapshots after reconnect.
- During failover, maintain a defined pause/hold behavior until source and timeline state are consistent.
- Keep event-wide admissions independent of B-zone instance capacity. Sandbox and Live usage accounting separated.

## Test proposals
- BRIDGE-001: A projection appears consistently in B1/B2 without duplicate authoritative performers.
- BRIDGE-002: Connected mode returns permitted B audience presence to A.
- BRIDGE-003: Broadcast mode does not expose B presence to A.
- BRIDGE-004: A authorized gimmick command reaches B1/B2 once per command ID.
- BRIDGE-005: B rejects unauthorized command or stale source epoch.
- BRIDGE-006: A1->A2 switch fences late A1 messages.
- BRIDGE-007: B reconnect reconciles event timeline and gimmick state.
- BRIDGE-008: B remains an independent seat/session authority.
- BRIDGE-009: high audience volume applies bounded audience projection to A.

## Open questions
- Connected-mode A visibility of audience: all B audience vs selected B zone vs sampled/aggregated audience.
- Whether Connected and Broadcast modes can coexist across B zones within one event.
- Whether A can initiate bidirectional physical interactions with selected audience, requiring cross-instance interaction authority.
- Live system repository and media protocol ownership.

## Decision update — 2026-11-01
- User selected C: selected B instances are connected bidirectionally to A; all B instances contribute aggregated audience reactions. Other B instances receive A broadcast/projection without direct audience projection to A.
- User requested optional distribution and synchronization of A-side gimmicks to B at connection/broadcast start.

## Proposed gimmick distribution and synchronization (not yet approved)
- `GimmickPackage`: versioned immutable manifest, hash, dependencies, required capabilities and execution permissions.
- `GimmickBinding`: maps abstract stage/lighting/audience targets from A to concrete targets in each B; supports different venue layouts.
- `GimmickState`: per-B authoritative execution state with event clock position and source epoch.
- `GimmickEvent`: authenticated command ID, source epoch, event timestamp, target binding, operation and parameters.
- On join: validate package/rights/capabilities -> distribute hash-verified package/assets -> bind to B -> apply snapshot -> replay missing events -> activate scheduled execution.
- During performance: A controls timeline and issues commands; each B validates, executes locally and ACKs; stale epochs and duplicate IDs are rejected.
- On A failover: reconcile source epoch and timeline before accepting new commands; old A cannot operate B.
- Initial PoC should synchronize declarative definitions and bounded commands. Arbitrary scripts must not execute in B without sandboxing, capability control and resource quotas.
- Package distribution does not automatically grant viewers access to paid/private assets; access policy and SansaSphere rights remain separate.

## Additional test proposals
- GSYNC-001: identical package hash is not downloaded again.
- GSYNC-002: mismatched hash or missing dependency blocks activation.
- GSYNC-003: A gimmick target binds to different B venue layouts.
- GSYNC-004: reconnect applies snapshot and missing events without duplicate effects.
- GSYNC-005: delayed event from old A source epoch is rejected.
- GSYNC-006: package lacking permitted capability cannot execute.
- GSYNC-007: connected and broadcast B receive consistent timeline effects while A only sees selected B audience.
