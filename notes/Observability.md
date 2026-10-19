---
tags: [gtm-os, observability, concept]
status: growing
created: 2026-10-19
updated: 2026-10-19
aliases: [logging, monitoring, three pillars]
---

# Observability

## Correction
"Logging" was being used for three different things. The umbrella term is **observability**: being able to tell what the system is doing from what it emits. It serves three goals: **monitoring**, **troubleshooting** and **improvement**.

## The three pillars
1. **Logs**: a detailed record of what happened, one entry at a time. → troubleshooting.
2. **Metrics**: numbers over time (runs per hour, failure rate, average duration). → monitoring and alerts.
3. **Traces**: one item followed across every step and worker. In GTM terms, one lead's journey through spam check → enrichment → CRM update. → troubleshooting across workers.

Plus a fourth thing people confuse with logging: the **business event log** ("lead scored", "reply received"). That isn't technical logging; it's *data*. See [[Two Log Streams]].

## You don't need metrics or a tracing tool yet
- **Metrics come almost free from structured logs.** Failure rate is just a count of entries with `status = error`. Decide good logs now; derive metrics later.
- **Tracing comes cheap from a [[Correlation ID]].** Searching one ID shows the whole journey, in order.

## What makes logging good
- **Structured, not free text**: every entry has the same fields (see [[Telemetry Module#Standard fields]]).
- **One shared correlation ID** across every step of a journey.
- **Levels**: info, warning, error, so noise stays apart from problems.
- **No secrets, and no unnecessary personal data.**
- **Alerts only on what someone will act on.** Otherwise people learn to ignore them.
- **Don't record everything.** Telemetry costs money to store and time to read. Keep detailed technical logs for weeks, not forever.

## Tiers
| Tier | What it is | Tools |
| --- | --- | --- |
| Out of the box | The orchestrator's built-in run logs + Slack alerts on failure | Trigger.dev / Inngest dashboards, Slack |
| Middle | Error tracking + searchable log storage | Sentry, Better Stack, Axiom |
| Fully engineered | Standard telemetry pipeline into a full observability platform | [[OpenTelemetry]] → Grafana, Datadog |

**Recommendation:** start at the first tier, plus a business event table in Supabase, with your own [[Telemetry Module]] so moving up a tier later changes nothing in the workers.

Related: [[System Design Principles]] (decision 5), [[Engine vs Core]].
