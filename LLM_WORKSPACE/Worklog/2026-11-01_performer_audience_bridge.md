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
