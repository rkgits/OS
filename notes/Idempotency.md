---
tags: [gtm-os, reliability, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [idempotent, upsert, idempotency key, twice equals once]
---

# Idempotency

**Doing the same thing twice has the same effect as doing it once.** Think of a lift button: press it five times and the lift still comes once.

## The rule
> **Assume every action can run twice, and make sure twice equals once.**

Repeats aren't rare edge cases; they're normal. They come from **retries** (the engine re-runs a failed task), **duplicate deliveries** (a webhook sender times out and sends again; [[Form Submission Contract#How the plugin delivers]]), **overlapping syncs** (the lookback window in [[Pull-Based Sync]]), and **double clicks** on a dashboard button.

## Correction: it's designed in, not a natural property
The instinct was: "adding a contact to HubSpot or to a campaign is idempotent, because once they're in, adding them again doesn't matter." **Not automatically.** Creating a HubSpot contact twice can create a **duplicate contact**; sending an email twice really sends two emails. Whether an action is safe to repeat depends on how it's built.

| Kind | Examples | Safe to repeat? |
| --- | --- | --- |
| **Naturally idempotent** | "Set the deal stage to Won", "set opt-out = true" | Yes: the end state is the same however many times it runs |
| **Not naturally idempotent** | "Create a contact", "send an email", "add 1 to a counter", "add to campaign" (on many tools) | No: each run adds something. **Needs protection** |

## How you protect it
1. **Upsert by a stable ID.** Store each record keyed by its source ID (e.g. Smart Lead's reply ID): **update if it exists, insert if it doesn't**. Seeing it again changes nothing.
2. **An idempotency key.** Attach a unique key to the request (e.g. the form's `event_id`) and refuse to process the same key twice. A database uniqueness rule makes this enforced, not hoped for.
3. **Check before a side effect.** For actions in the outside world (emails, CRM writes): check for a "done" record first (e.g. a `notified` business event), act, then write the record ([[Source Routing#Side effects]]).
4. **Prefer "set" over "add".** "Set status = X" is naturally safe; "increment by 1" is not.

## How to say it
"**Every operation that writes to an external system must be made idempotent**, using an idempotency key, an upsert or a check-before-act."

## Where it shows up in this vault
- The receiver deduplicates on `event_id` ([[Webhook Ingress]]).
- Lifecycle business events are unique per submission ([[Two Log Streams#The four v0 events]]).
- The sync's overlap window is only safe because writes are upserts ([[Pull-Based Sync]]).
- Future dashboard actions (add to HubSpot, add to a campaign) must each be made idempotent before they ship ([[System Map]], layer 4).

Related: [[Recurring Lessons]].
