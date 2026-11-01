# Concert Capacity Research / Design Worklog — 2026-11-01

## Scope and status
This is a non-canonical research/worklog note for SansaXR-WorldCore develop, aligned with HLDocS v0.6.1-preview. Numerical load targets and partition policy below are **proposals**, not approved specifications or demonstrated performance.

## Primary-source venue capacity references
- Zepp DiverCity Tokyo: standing 2,473, seated 1,102. https://www.zepp.co.jp/hall/divercity/
- Yokohama Arena: maximum 17,000. https://www.yokohama-arena.co.jp/organizer/equipment/
- GMO Arena saitama stadium configuration: center-stage approx. 37,000, end-stage approx. 30,000. https://www.saitama-arena.co.jp/business/layout_sizes/layout_sizes_stadium/
- Tokyo Dome: maximum 55,000. https://www.tokyo-dome.co.jp/business/event-space/
- Nissan Stadium: 72,327 spectator seats. https://www.nissan-stadium.jp/shinyoko-park/stadium.php
Venue capacities are **not** measurements of simultaneous networked VR participants, and show-dependent actual attendance differs.

## Proposed system goals (unapproved)
- Load test scenarios: 100, 3,000, 20,000, 55,000, 75,000 concurrent audience members per event.
- Do not equate event-wide population with single simulation instance capacity.
- Distinguish: event admissions, connected sessions, active avatars, nearby replicated entities, media subscribers, and audience-zone occupants.
- Define an Event Coordinator with shared event clock/timeline and stable event ID, while independent audience zones/instances handle local presence and proximity interaction.
- The live media service is a separate capability provider; do not assume one server distributes individual media streams to 75,000 clients.
- Admission should reserve capacity atomically per zone; use idempotent request IDs, lease expiry, queue/deny on saturation, and avoid double count across handoff.
- For user mobility, preserve event time and entitlement while transferring zones. Global performance depends on density, update frequency, bandwidth, media distribution, and per-zone complexity.
- Usage accounting to SansaSphere must avoid double counting when users cross zones and must separate sandbox/test activity.

## Solo implementation sequencing (requires user approval)
1. Specify the admission/capacity domain model with fake entitlement and fake live capability provider.
2. Document numbered test cases for boundary, capacity reservation, duplicate admission, queueing, transfer, disconnect and event-wide metrics.
3. Implement Rust reference only after specifications and tests are accepted.
4. Load-test admission/state management separately from full XR rendering, live media delivery and real-world-scale networking.

## Open design decision
Should event capacity be a configurable event-wide limit independent of automatically scaled audience-zone capacity? Recommended yes; choose initial zone sizes only from benchmarks, not venue seating counts.

## Cross-repo dependencies
SansaSphere: paid admission, usage aggregation, payout. SansaXR: avatar rendering, local physics, proximity audio. Live system: media distribution and clock sync. Keep external interfaces mockable.
