---
tags: [gtm-os, observability, data-model, decision]
status: growing
created: 2026-10-19
updated: 2026-10-21
aliases: [technical logs vs business events, event log]
---

# Two Log Streams

**Decision: split by audience.**

| | Technical logs | Business events |
| --- | --- | --- |
| Answers | Is the *system* healthy? | What happened to our *leads and accounts*? |
| Read by | Whoever maintains the system | The GTM team, managers, dashboards |
| Shape | Detailed, noisy, changes with the code | Simple, stable names ("lead enriched", "reply received") |
| Retention | Short (weeks) | Long-term |
| Access | Limited (may hold sensitive detail) | Broad |
| Lives in | The orchestrator's logs / a log tool | **One table in the core Supabase database**, next to people and accounts |

## Correction
Don't build manager-facing visuals on top of technical logs. They're noisy, unstable and can be sensitive. Manager visuals belong on business events, which are designed to be read, counted and charted.

## Clarifications
- The business event table is **not a separate database**. It's one table in the core Supabase database: the event log called "the backbone" in [[System Design Principles]].
- For business events, Supabase is both the **store** and the source for the **view** (the dashboard), which is why events must be clean and stable.
- The two streams share one thing: the [[Correlation ID]]. When a manager asks "why did this lead never get an email?", you jump from their business event straight to the technical trail.
- Both streams are written through the same [[Telemetry Module]].

## The four v0 events
**Decision:** the first worker ([[Form Intake Pipeline]]) writes four stable events. Each records a *fact that happened*; the **outcome goes in the data, not in the event name**.

| Event type | Written when | `data` |
| --- | --- | --- |
| `form_submission.received` | the receiver stores the row | form, source |
| `form_submission.classified` | the task decides | decision (`spam` / `accepted` / `needs_review`), score, reasons, rules version, form |
| `form_submission.notified` | the result email is sent | channel, recipient role |
| `form_submission.failed` | a step fails for good after all retries | step, short error |

**Why one `classified` event, not `spam_detected` and `lead_accepted`:**
- A dashboard counts outcomes with one query: the event type, grouped by `data.decision`.
- Adding a decision later (say `needs_review` becoming two states) needs no new event name, so the stream stays **stable** while the rules change. The rules version in the data says which rules decided.

**Shape rules**
- Every event carries the [[Correlation ID]], an **actor** (who did it: receiver, classifier, notifier), and the **subject** (what it's about: the submission).
- The engine's run ID is an **extra field in the data**, never the primary trace ID.
- **No personal data in `data`**: no email, name, message, IP or user agent. Redaction in the [[Telemetry Module]] enforces it.
- **Test events write none** of these.
- **Lifecycle events are unique per submission** (a database rule), so a retried run can't record the same fact twice. `failed` may repeat on purpose; a replayed run that succeeds simply adds the missing facts after it.

**Later, not now:** a `reviewed` event (a human overrides a verdict, which is how rules get tuned), `lead.created` once identity resolution exists ([[Identity Resolution]]), and `reprocessed` for stuck rows.
