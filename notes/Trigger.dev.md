---
tags: [gtm-os, ingestion, tool]
status: growing
created: 2026-10-18
updated: 2026-10-18
aliases: [Trigger]
---

# Trigger.dev

A platform for **background jobs**: work that runs with nobody watching a screen. Open source, Y Combinator-backed, popular with TypeScript developers. Much smaller than n8n or Zapier. Being explored, not committed to.

## Core concepts
- **Tasks**: units of work written as TypeScript functions.
- **Triggers**, the things that start a task:
  1. **Schedule** → polling.
  2. **Event** (webhook) → push.
  3. **Another task** → chaining (spam check → domain research → LinkedIn lookup).
- **Retries** on failure; **queues / concurrency limits** to stay under vendor rate limits; **run logs** for every run.

## Day to day
- Tasks live in your own code project; test locally with their dev command, deploy, and they run on their cloud.
- **Long-running and pausable**: a task can "wait one hour, then check for a reply", which suits enrichment chains.
- **Idempotency keys**: "run once per lead", so a duplicate webhook doesn't enrich someone twice.
- **Web dashboard** of every run, step, input, output and failure. This is the cure for fragility.

The mental shift from n8n: **the code is the workflow; Trigger.dev is the reliable engine underneath.** No drag-and-drop canvas.

## Pricing model (approximate, Oct 2026; verify on their pricing page)
- Plans include monthly credit: Free ≈ $5 usage; Hobby $10/mo incl. $10; Pro $50/mo incl. $50.
- Each run costs a tiny fixed fee **plus compute seconds**. It's billed on time, not "tokens".
- The per-second rate scales with machine size; most connectors fit the smallest machine.
- Paused/waiting tasks are put to sleep, so idle time is mostly free.
- Implication: a connector that runs for a few seconds costs very little even at thousands of runs, unlike n8n's per-execution billing.

## Alternatives
- **Inngest**: very similar model.
- **Temporal**: heavier, enterprise-grade durable execution.

How to use it without being locked in: [[Engine vs Core]]. Where it fits: [[Ingestion Layer Options]].
