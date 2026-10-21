---
tags: [gtm-os, build, ingestion]
status: growing
created: 2026-10-19
updated: 2026-10-21
aliases: [website form webhook, first build]
---

# Form Intake Pipeline

**The first real build:** website form submissions (Galliot Forms, a custom WordPress plugin on galliot.us) → spam decision → business events. Everything else gets built around the patterns proven here.

## Status
**v0 is built and live on production** in `galliot-us/GTM-Engine` (ticket: [issue #1](https://github.com/galliot-us/GTM-Engine/issues/1); setup, deploy steps and known gaps are in that repo's README, not here).

```
WordPress plugin → receiver (Supabase Edge Function) → Trigger.dev task → result email + saved decision
```
- **Receiver:** checks a URL token, stores the raw row, writes `received`, starts the task, answers `202` immediately. A repeated `event_id` is answered `duplicate` and not processed again.
- **Task:** loads the row, classifies it with plain functions in the core, saves the decision, writes `classified`, emails the verdict, writes `notified`. After all retries fail it writes `failed` and sends an alert email.
- **Verified end to end** on Oct 19: a spam case, a legitimate case, a duplicate, a test event, a real contact submission from galliot.us, and a forced failure.

> [!warning] Correction
> The assumption "the plugin doesn't sign requests" was wrong. It can sign (`X-Galliot-Signature`, HMAC over `timestamp.body`); the receiver does not verify it yet, so a URL token is the only check. See [[Form Submission Contract#Plugin signing: it exists, and it's its own scheme]].

## Correction
"Build on top of Trigger.dev". The phrase should be **build *on* it, not around it** ([[Engine vs Core]]). In v0 this is concrete: spam rules, the contract and email formatting are plain TypeScript with no Trigger.dev imports; the task is a thin wrapper; Gmail and Supabase sit behind small adapters.

## Design
Trigger.dev doesn't expose a public URL that WordPress can post to; tasks are started from your own code. So a small **webhook receiver** sits in front. That's good news, because the receiver is where the core does its job ([[Webhook Ingress]]):

1. **Verify** the request came from the site. *v0: a secret URL token only; signature verification is the next step.*
2. **Deduplicate** on the source `event_id` (**idempotency**): a retried delivery is processed once.
3. **Mint the [[Correlation ID]]**.
4. **Save the raw submission** untouched to a staging table, *before* any parsing.
5. **Trigger the spam-check task** in the orchestrator and **answer WordPress immediately**. Never make the website wait for slow work.

The task then decides spam vs not, and writes business events ([[Two Log Streams#The four v0 events]]): `received` → `classified` (decision in the data) → `notified`, or `failed`.

## The spam decision
The plugin *supplies* signals; the **core decides**, so rules change without redeploying WordPress.

Same layered pattern as [[Identity Resolution]]:
- **Simple rules first** catch the obvious cases.
- **An LLM judges only the grey zone.**
- **Two thresholds + a review queue.**

v0 has the first and third: weighted rules (`rules-v0`) summed into a score, **two thresholds** (60 and up = spam, 30 to 59 = needs review, below 30 = accepted), and every fired rule recorded as a reason. The LLM judge and the review queue don't exist yet, so `needs_review` is only a label. The weights are provisional; they get tuned against real traffic.

> [!example] Test case (real email, Oct 18)
> 2-character message ("Hy"), a free Gmail address containing "seoexpert", a two-year-old browser (Chrome 109), from Pakistan to a US consulting site, **captcha passed**. Almost certainly spam. That's why captcha alone isn't enough. v0 scores it 70 (**spam**) from three weak signals; none alone would be enough.

## Lessons from the first real submission
1. **Store first, parse second paid off immediately.** The first real contact form failed validation (an empty `utm` arrives as `[]` from PHP, not `{}`). Because the raw row was already stored, nothing was lost: the failure was recorded as an event, an alert went out, and the run was **replayed** after the fix. See [[Form Submission Contract#Quirks the real traffic exposed]].
2. **Test the failure path with a deliberately broken input**, not only the happy path. A broken payload that the receiver accepts but the task rejects exercised retries, the `failed` event, the row status and the alert in one go.
3. **Silent failure is the real risk, not failure.** A failed event in a table that nobody reads is invisible. The task now emails an alert when a run fails for good.
4. **An alert channel that shares a dependency with the thing it watches can fail with it.** Result emails and failure alerts use the same Gmail login, so a Gmail outage hides both. A second, independent channel (the orchestrator's own alerts) covers that.
5. **The receiver is the part that's hardest to observe.** It runs in a different runtime (Deno) from the workers (Node), so the shared [[Telemetry Module]] doesn't reach it. See [[Telemetry Module#Where v0 stops]].
6. **Setting a webhook URL replaces email delivery** in the plugin. Switching a live site over is therefore a real change; the safe order is: test event, then one real submission, with a one-step rollback (clear the URL).

## Decisions
- **Notification path for v0: email**, through a replaceable adapter (Gmail today). It's the same path the owner already watched when the plugin emailed submissions.
- **Alerts go to a plus-address** of the owner's inbox, so they can be filtered without a second mailbox.
- **Test events are stored and classified but write no business events**, and the result email is labelled as a test.
- **Technical logs and event data carry IDs and statuses only**, never visitor data; redaction is enforced in the telemetry module, not left to each worker.

## Personal data
IP, country, user-agent and time zone are personal data (e.g. under GDPR). They're fine in the raw staging table with limited access (row-level security on, service role only); the [[Telemetry Module]] keeps them **out of technical logs** and out of event data.

Why the receiver sits outside the engine: [[Webhook Ingress]]. Adding more sources: [[Source Routing]].

Related: [[Form Submission Contract]], [[Ingestion Models]].
