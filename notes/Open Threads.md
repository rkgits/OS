---
tags: [gtm-os, todo]
status: growing
created: 2026-10-15
updated: 2026-10-19
---

# Open Threads

- [ ] Read the Galliot Forms webhook code (in `galliot-us/galliot-website`?): signing, retries, real payload ([[Form Submission Contract]]).
- [ ] Build [[Ticket - Form Intake v0]]: receiver + Trigger.dev task + rule-based spam decision, including capturing a real **Send test event** delivery.
- [ ] Define v1 of the contract: CloudEvents envelope + Standard Webhooks headers + version field.
- [ ] Write the first spam rules and pick the two thresholds.
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
- [x] Which hurts more, n8n cost or fragility? → Both, so move to a code-first orchestrator ([[Ingestion Layer Options]]).
- [x] Choose a tool vs build around it → [[Engine vs Core]].
- [x] What's a good logging standard? → [[Observability]], [[Two Log Streams]], [[Telemetry Module]].
- [x] Who creates the job / event ID? → the core mints a [[Correlation ID]]; the engine's run ID is an extra field.
