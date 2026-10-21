---
tags: [gtm-os, observability, component]
status: growing
created: 2026-10-19
updated: 2026-10-21
aliases: [structured logger, logging wrapper, observability SDK]
---

# Telemetry Module

**Telemetry** means "measurement from a distance". Picture a Formula 1 car streaming speed, temperature and tyre wear back to the pit crew. The workers are the car; you're the pit crew.

## Correction: core, not engine
Logging belongs in the **core** (what you own), not the **engine** (the replaceable orchestrator). See [[Engine vs Core]]. Logging is a **cross-cutting concern**: something every worker needs. You don't let each worker do it its own way. You build **one small shared module** that every worker calls, so fields, IDs and format are always consistent. The engine's own logs are a bonus underneath; swap engines and this module doesn't change.

## Spec (small on purpose)
1. **Start or continue a trace**: reuse an incoming [[Correlation ID]], or mint a new one.
2. **Fill standard fields automatically**, so the worker never has to remember them.
3. **Simple functions**: `info`, `warn`, `error` for technical logs, plus one `event` call for business events (written to the Supabase table; see [[Two Log Streams]]).
4. **Redact secrets** (API keys, tokens) and unnecessary personal data before anything is written.
5. **Never crash the worker.** If logging fails, the real job still finishes.

Roughly a hundred lines of code; an AI assistant can write it in an afternoon. The thinking is the hard part.

## Standard fields
time · worker name · correlation ID · engine run ID · lead / account ID · action · status · duration · error message (if any)

## The telemetry pipeline
1. **Emit**: workers produce data through this module. ← *the only stage you own*
2. **Collect**: something gathers it from all workers.
3. **Store**: it lands somewhere searchable (the Supabase business event table is a *store*).
4. **View**: dashboards, searches, alerts.

Stages 2–4 are swappable tools. Make the module speak [[OpenTelemetry]] and they stay swappable.

## Names engineers would recognise
- **Logging wrapper / structured logger**: your thin layer over a standard library (TypeScript: **Pino** or **Winston**).
- **Context propagation**: passing the correlation ID along.
- **Instrumentation**: adding this logging to the workers.
- **Telemetry module / observability SDK**: the module covering logs and events together.
- **Event tracking**: the business-events part on its own.

## v0 implementation
The first worker uses a small module that follows the spec above: it reuses the incoming correlation ID, fills standard fields (time, worker, correlation ID, run ID), offers `info` / `warn` / `error` plus one `event` call, **redacts** secrets and personal data before anything is written, and **never throws**. Technical logs go to the orchestrator's run log; business events go to the Supabase event table through a small adapter, which keeps the module itself free of any engine or database code ([[Engine vs Core]]). Redaction and "never throws" have tests.

## Where v0 stops
- **The receiver is outside it.** The module is Node code; the receiver runs in a different runtime (Deno), so the first hop of every journey logs as plain text without the standard fields. Open design question: one shared module that runs in both, or two thin implementations of the same field contract. See [[Open Threads]].
- **Standard fields are partial.** Time, worker, correlation ID and run ID are automatic; duration, status and action are not.
- **Alerting is a separate concern.** Telemetry records a failure; something else must tell a human. v0 emails an alert when a run fails for good, through the same mail login as the results, so a mail outage would hide both ([[Form Intake Pipeline#Lessons from the first real submission]]).

Related: [[Observability]].
