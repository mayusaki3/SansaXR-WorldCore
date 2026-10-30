# Dedicated Runtime Instance Requirement — 2026-10-30

## User clarification
A live concert/event may require its own live system. Therefore entry to a dedicated instance is determined not only by access control but also by the runtime capabilities the content needs.

## Architectural refinement
- Keep physical seamless-world entrance and venue exterior in the shared world where possible.
- Dedicated instance selection is based on runtime capability requirements, isolation, synchronization domain, capacity and resource policy, in addition to access policy.
- Paid content does not automatically require a separate instance; free content can require one.
- Dedicated instance is a logical runtime isolation unit, not necessarily a dedicated machine/process.
- A live venue may use audio/video streaming, performer/stage controls, audience synchronization, event-specific gimmicks, admission checks, and usage events.
- SansaXR-WorldCore should define instance lifecycle, capability requirements, admission and transition contract; implementation of specialized live system belongs to a separate module/service with a yet-to-be-decided repository boundary.
- Existing WorldDefinition / WorldInstance / AccessRegion / Portal / WorldSession distinction remains.
- Transition authorization and capacity/resource reservation must succeed before committing migration; failure retains the source session.
- Sandbox usage events and Live usage events remain separated for SansaSphere revenue evaluation.

## Proposed data additions (not yet approved as formal specification)
- RuntimeProfile: required capabilities, isolation policy, capacity, synchronization mode.
- InstancePlacementPolicy: shared, isolated, dedicated runtime; placement is orthogonal to paid/private access.
- CapabilityProvider: optional interface for live-system readiness and health.
- InstanceAdmission: entitlement + runtime readiness + capacity + venue policy.
- InstanceLifecycle: provision, ready, admit, drain, stop, recover.

## Solo PoC test candidates
- INST-CAP-001: paid facility without special runtime remains in shared instance.
- INST-CAP-002: free live event requiring a live system routes to a dedicated instance.
- INST-CAP-003: missing live capability blocks transition without losing source session.
- INST-CAP-004: insufficient capacity rejects or queues admission without partial migration.
- INST-CAP-005: reconnect after committed transition restores exactly one session.

## Coordination later
- SansaSphere: entitlement, usage measurement, revenue-sharing attribution for events.
- SansaXR: presentation of portals and live audio/video clients.
- Specialized live-system implementation: capabilities and deployment API; repository undecided.

## Status
Design clarification recorded in Worklog; no runtime implementation performed.
