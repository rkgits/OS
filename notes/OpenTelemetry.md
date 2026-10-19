---
tags: [gtm-os, observability, tool, standard]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [OTel]
---

# OpenTelemetry

**Both a standard and a set of libraries.**
- **Standard**: an agreed, vendor-neutral format for logs, metrics and traces.
- **Libraries**: free implementations of that standard for each programming language.
- Open source, under the Cloud Native Computing Foundation (CNCF); very widely adopted.

## Why it matters here
It's [[Engine vs Core]] applied to monitoring. If the [[Telemetry Module]] emits OpenTelemetry, the data can go to Sentry today and Grafana next year **without touching the workers**.

## Correction
You're not "building an engine inside Trigger.dev". You're **instrumenting your own code** with the OpenTelemetry library, inside your telemetry module. Because that code belongs to the core, it moves with you to any engine, and any backend that speaks the standard can receive the data.

## Trigger.dev and OpenTelemetry
As far as we know, [[Trigger.dev]] records its own run traces and logs with OpenTelemetry and lets you add instrumentation and export to other tools. **Unverified; check their docs before relying on it.**

> [!warning] Don't lean on it
> Even if the engine's logs already use OpenTelemetry, your module still owns the [[Correlation ID]] and the standard fields.
