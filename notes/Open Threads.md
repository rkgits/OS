---
tags: [gtm-os, todo]
status: growing
created: 2026-10-15
updated: 2026-10-19
---

# Open Threads

Learning and design questions only. Build work lives as issues in [GTM-Engine](https://github.com/galliot-us/GTM-Engine/issues).

- [ ] Fair comparison note: Trigger.dev vs Inngest vs Temporal vs self-hosted n8n vs fully custom.
- [ ] Design one example connector (e.g. Smart Lead) engine-agnostically, following [[Engine vs Core]].
- [ ] A shared connector pattern so custom connectors don't become twenty one-offs (common interface: auth, incremental cursor, pagination, raw landing table).
- [ ] Per-source inventory of all 20–40 systems ([[Per-Source Decision Framework]]), including Airbyte coverage.
- [ ] Design the **business event table**: which events exist, what each records, how the dashboard reads it ([[Two Log Streams]]).
- [ ] Verify how Trigger.dev uses [[OpenTelemetry]] in its docs.
- [ ] How the match score is built ([[Identity Resolution]]).
- [ ] Real-time vs good-enough tradeoffs, source by source.
- [ ] Moving from visualization to **action** from the dashboard, without welding the insight and action planes together.

## Resolved
- [x] First build → [[Form Intake Pipeline]] v0 built ([GTM-Engine #1](https://github.com/galliot-us/GTM-Engine/issues/1)).
- [x] Which hurts more, n8n cost or fragility? → Both, so move to a code-first orchestrator ([[Ingestion Layer Options]]).
- [x] Choose a tool vs build around it → [[Engine vs Core]].
- [x] What's a good logging standard? → [[Observability]], [[Two Log Streams]], [[Telemetry Module]].
- [x] Who creates the job / event ID? → the core mints a [[Correlation ID]]; the engine's run ID is an extra field.
