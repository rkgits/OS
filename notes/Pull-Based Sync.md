---
tags: [gtm-os, ingestion, concept]
status: seed
created: 2026-10-28
updated: 2026-10-28
aliases: [incremental sync, cursor, pagination, rate limits]
---

# Pull-Based Sync

Everything built so far was **push**: a website calls you ([[Webhook Ingress]]). But Smart Lead, HeyReach and HubSpot mostly need **pulling**: you call their API on a schedule. That's where most of the 20–40 tools live.

**The shift:** with a webhook, the source decides *when* you hear about something. With a pull, **you** decide, so you take on four problems that push hid from you.

1. **What has changed since last time?** Don't re-download all of HubSpot every hour. Store a bookmark ("last modified at", or a page token) and ask only for what's newer. The bookmark is a **cursor**; syncing this way is **incremental sync**. Downloading everything is a **full refresh**.
2. **Pages.** The API returns data a page at a time (say 100): **pagination**. If the sync crashes on page 37, does it restart from page 1 or carry on?
3. **Speed limits.** The API limits how fast you can ask: a **rate limit**. Go over it and you get HTTP 429. You **back off** and slow down.
4. **Deletes are invisible.** A contact deleted in HubSpot never shows up in "modified since X"; it simply stops appearing. **The trap that catches almost everyone.**

## Exercise (unanswered)
> [!question] The cursor trap
> A Smart Lead sync runs every 15 minutes and saves **"last run time"** as its cursor. A run starts at 10:00, takes 4 minutes, and saves **10:04**. A reply arrives at **10:02**, but Smart Lead only writes it to their database at **10:05**. **Does the next sync pick it up? Why or why not?**

> [!tip]- Answer (open after you've tried)
> **It's at risk of being missed.** The reply's "modified" timestamp is likely 10:02, and the next sync asks for "modified after 10:04". Two mistakes combine:
> 1. **The cursor came from your clock, not their data.** Use the highest `updated_at` you actually *received* as the cursor, not when your job ran.
> 2. **No allowance for late data.** Records can become visible after their timestamp (**late-arriving data**, replication lag). Re-read a small **overlap window** each run (e.g. cursor minus 10 minutes) and make writes **idempotent upserts**, so re-reading the same record is harmless.
>
> Plus the nightly **full reconciliation** from [[Ingestion Models]] as a backstop, which also catches deletes.

Related: [[Per-Source Decision Framework]] (rate limits and incremental sync as supply-side questions), [[Ingestion Layer Options]].
