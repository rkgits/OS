# GTM Unified Dashboard — Architecture Notes

Started Oct 15, 2026 · Last updated Oct 18, 2026 · Reza

*Living document: updated as the architecture discussion continues.*

A working brief on how to architect a holistic analytics dashboard that pulls data from many fragmented go-to-market systems (Clay, Smart Lead, HeyReach, CRM) into one place for visualization and, eventually, action.

## 1. The core problem, named

The overarching discipline is **data integration**. The thing you are building is a **data pipeline**: extract data from many sources, land it somewhere central, transform it into clean reporting tables, and visualize it.

Two classic pipeline patterns:

- **ETL** — Extract, Transform, Load. Clean and reshape the data *before* it lands.
- **ELT** — Extract, Load, Transform. Dump raw data in first, transform later with SQL. This is the modern warehouse approach and the one you would use.

## 2. The three integration approaches

### Approach A — Query the sources live (federated / query virtualization)

Each system stays the source of truth; you query it on demand.

- **Pros:** no duplication, always fresh (zero staleness); no storage cost or governance headache from copying personal data; fastest and cheapest to stand up, so it earns its keep as a proof of concept.
- **Cons:** **tight coupling** — your dashboard is bound to every vendor's API, rate limits, schema changes and uptime; one deprecated endpoint and you are down. **No historical state / no point-in-time analysis** — you only ever see what is true now, so you cannot ask what the pipeline looked like last quarter. That alone usually disqualifies it for a management dashboard, which lives on trends.

### Approach B — Push everything into the CRM (CRM as hub / system of record)

All data flows into HubSpot or Salesforce.

- **Pros:** one authoritative **system of record** the team already trusts and acts in; collapses the operational and analytical worlds into one place, so the system that shows the data can also act on it (the **operational system**).
- **Cons:** **impedance mismatch** — CRMs model the world as contacts and deals, so rich campaign and engagement data gets flattened or does not fit; **not an analytics engine** — heavy aggregations get slow and expensive, and you can hit API limits just reading your own data back out; plus vendor lock-in and write-quota limits.

### Approach C — Dedicated central database / warehouse (recommended)

Land all raw data in one central store, then transform it with SQL into clean reporting tables.

- **Pros:** **full history** (nothing is overwritten, so point-in-time works); **decoupled** — the dashboard only ever talks to your warehouse, not twenty vendor APIs; heavy analytics run without touching sources; becomes your **single source of truth for analytics**, with **staging** (raw) versus **transformed** layers.
- **Cons:** you now own a pipeline and must keep it fed and correct — the rest of this doc.

## 3. Choosing the store: Airtable vs Supabase vs Snowflake

These are **not the same category**.

| Store | What it really is | Good for | Ceiling |
| --- | --- | --- | --- |
| Airtable | Spreadsheet-database hybrid | Small, human-managed data; quick views | Tens of thousands of rows; not a serious analytical backend |
| Supabase | Hosted **Postgres** — a real relational transactional database (**OLTP**) | Developer-friendly, cheap application backend | Millions of rows, comfortably |
| Snowflake | A true analytical warehouse (**OLAP**) | Scanning and aggregating billions of rows fast | Built for scale you likely do not have yet |

**Recommendation: start on Supabase (Postgres).** At twenty to forty GTM systems your volume is millions of rows, not billions, and Postgres handles that comfortably while staying simple and cheap. Reach for Snowflake only when aggregations start to crawl. Do not over-engineer for a scale you do not have yet.

## 4. OLTP vs OLAP (the distinction to carry everywhere)

- **OLTP — Online Transaction Processing.** Many tiny writes and reads, one record at a time (a lead created, an email status flipped). The operational workhorse. Needs **ACID** guarantees (a transaction fully completes or fully rolls back). Stores data **row-oriented**, so grabbing or changing one full record is fast. → Postgres, MySQL, SQL Server, Supabase.
- **OLAP — Online Analytical Processing.** Few huge reads sweeping across millions of rows ("average lead score by channel over six quarters"). The analytical engine. Stores data **column-oriented**, so summing one field across everything flies. → Snowflake.

The real differentiator is not the query language — all speak SQL and all look relational. It is the **physical storage and execution strategy**: row storage for operational work, column storage for analytical work. OLAP versus OLTP is a characterization layer sitting on top of the shared relational model.

**Historical note:** twelve years ago OLAP often meant pre-computing rigid **data cubes** overnight (slice, dice, drill down, roll up) because live querying was too slow — for example SQL Server Analysis Services. Modern columnar warehouses got fast enough that the explicit cube largely melted away; you query the raw detail directly.

### The key mental correction

**Never pick the engine from how the data arrives. Pick it from the questions you ask and the volume you ask them against.**

Your events are born transactionally (OLTP-shaped writes), but a dashboard asks analytical questions (OLAP-shaped reads). Analytical purpose would point at a column store — but your volume is small enough that a row-oriented Postgres serves it fine. So: analytical in purpose, modest in scale, therefore use Postgres and do not pretend you are Netflix.

## 5. Ingestion: getting data in

First axis to name: **push vs pull**. In a pull model your system asks each source "what is new?" on a schedule. In a push model the source tells you the instant something happens.

### Model 1 — Scheduled batch pull (polling)

On a timer, call each API, grab what changed, load it.

- Pro: simple, predictable, easy to debug.
- Con: latency (data is as stale as the last run); wasted quota asking "anything new?" and hearing no.

### Model 2 — Webhooks (push)

The source fires an HTTP call at you the instant something happens.

- Pro: near real-time, no wasted polling.
- Con: you must run an always-listening endpoint and handle the messy realities — **idempotency** (a repeated event must be harmless) and **retries** (what if you are down when it fires).

### Model 3 — Hybrid (recommended)

Webhooks for events that must feel live (a hot lead, an inbound reply); scheduled batch for the slow-moving bulk (nightly campaign stats). Match each source to its natural cadence.

**Critical additions:**

- Many GTM tools (Clay, Smart Lead, HeyReach) have **patchy webhook coverage** — the vendor's capabilities often decide for you, not your elegant latency reasoning.
- Even when a webhook exists, keep a **periodic batch pull as a backstop** — webhooks silently drop events during outages. The pattern: webhooks for freshness, nightly batch to reconcile.
- **Do not let the analytics layer become the thing taking business actions.** Keep your **system-of-insight** separate from your **system-of-action**. An auto-reply to a lead should live in the outreach tool's own automation, not be triggered by your reporting warehouse.

## 6. The per-source decision framework

For every system you integrate, assess two sides, then the ingestion model picks itself.

**Demand side (what you need):**

- **Freshness requirement** — how stale can this data be before a decision suffers?
- **Criticality** — how much damage if it is wrong or missing?
- These are two different axes. A hot inbound reply is high-freshness. Quarterly campaign cost is low-freshness but high-criticality. Do not collapse them into "importance."

**Supply side (the vendor's integration surface — interrogate all three, not just webhooks):**

- Do **webhooks** exist, and for which specific events? Partial coverage is the norm.
- Can you do **incremental sync** — pull only records changed since a timestamp (the richer form is **change data capture**)? If not, you are forced into full re-pulls every run.
- **Rate limits and pagination** — the unglamorous killer. A vendor can let you pull everything and still throttle you so hard that a nightly sync takes six hours. People discover this in production, never in planning.

## 7. Identity resolution (unifying records)

To see everything about one person across email, LinkedIn, GitHub and the rest, you need **identity resolution** (a.k.a. **entity resolution**). Collapsing duplicate records is **deduplication**; the result is a **unified customer profile**; the structure linking it all is an **identity graph**.

**The goal:** every record about a person points to one **canonical internal ID** that you mint and control — never a vendor's ID.

**The hard part is not assigning the number — it is deciding which records deserve the same number.** There is rarely one clean key (email breaks when work and personal differ; names collide and are misspelled). So use a hierarchy of matching rules:

1. **Deterministic matching** — an exact strong key (verified email, LinkedIn URL). High confidence.
2. **Probabilistic matching** — when that fails, score similarity across weak signals (name plus company plus city) and match above a confidence threshold.

**Non-negotiable practice:** keep the raw source records intact and build the unified identity as a separate layer on top, linking back. Never merge in place — your matching will be wrong sometimes, and fusing two different people is a horrible, hard-to-detect bug you must be able to unwind.

### Person vs account (B2B)

Model it as a two-layer hierarchy, a one-to-many relationship:

- **Person** (bottom) — the individual human (contact / lead). Every actual touch happens here; you cannot email a company.
- **Account** (top) — the company. One account, many people. Revenue closes here, so you report primarily at this level.

**Resolve identity at the person level first, then roll people up to accounts.** Account-level metrics ("how engaged is Acme") are aggregations of person-level events — you need the fine grain underneath for the coarse grain to mean anything. "Accounts matter, not people" is a dangerous half-truth.

**Watch out — people change companies.** A person's account link must change over time without erasing what they did at the old company. That is a **slowly changing dimension**; ignore it and your historical account numbers quietly rot.

## 8. The ingestion framework (how the pipes actually get built)

### Two corrections first

**The dashboard must not call vendor APIs.** Having the dashboard pull from Smart Lead, HeyReach and the rest on demand is Approach A again, the live-querying model already rejected. It ties the display layer to twenty vendors' uptime. The principle is **separation of concerns**: the dashboard reads only from Supabase, and a separate **ingestion layer** keeps Supabase filled.

**Your n8n flows are two different jobs wearing one coat.**

- **Extraction**: copying a vendor's records into your database (Smart Lead's campaign stats, HeyReach's message events). Dumb, repetitive, high-volume.
- **Enrichment orchestration**: a multi-step workflow that reasons about one record (a sign-up arrives → spam check → domain research → LinkedIn lookup → write the result). Low volume, many steps, branching logic.

Running both through a tool that bills per execution is part of why the n8n bill hurts: extraction generates huge volumes of trivial executions.

### The four options

1. **Managed ELT connectors** (Airbyte, Fivetran). A library of ready-made pipes. Each already knows how to talk to one vendor. You give it credentials, point it at Supabase, set a schedule, and it copies new and changed records itself, handling pagination, incremental sync and retries. No code. *Weakness:* the library covers popular tools; niche GTM tools like HeyReach are often missing.
2. **Self-hosted n8n.** The same n8n you know, run on your own server. The community edition is not billed per execution, so cost becomes a flat server bill. *Trade-off:* you now look after that server, and n8n is still a poor fit for bulk extraction.
3. **Code-first orchestrator** (Trigger.dev, Prefect, Dagster). You write each connector as a small function ("call this API, take the results, save them"). The orchestrator is the manager around your code: it schedules runs, retries failures and keeps logs.
4. **Fully custom** (Supabase Edge Functions scheduled with pg_cron). The same small functions, run by you. Cheapest, but with no manager around it, every failure is yours to notice and handle.

**Recommendation: hybrid.** Managed connectors (Airbyte) wherever one exists; a code-first orchestrator for everything else, including the enrichment workflows.

### Trigger.dev in detail

Trigger.dev is a platform for **background jobs**: work that runs behind the scenes with nobody watching a screen. You write the work in TypeScript as units called **tasks** (e.g. "pull yesterday's Smart Lead stats and upsert into Supabase").

What starts a task is its **trigger**:

1. **A schedule** (every night at 2 a.m.) → this is your polling.
2. **An event** (a webhook when someone joins the subscription list) → this is your push.
3. **Another task** → chaining; spam check → domain research → LinkedIn lookup becomes one workflow.

The safety net it provides around your code:

- **Retries**: a failed vendor call is retried automatically.
- **Queues / concurrency limits**: cap how many tasks run at once, which keeps you under vendor rate limits.
- **Run logs**: every run is recorded, so you can see exactly which step broke.

**Cost model (verify on their pricing page; these change):** billed mainly on compute time used, not per step as n8n is. Open source, so self-hosting is possible.

**The honest catch:** no drag-and-drop canvas. You write code, realistically with an AI coding assistant doing much of it.

### Still to decide

- Which hurts more today: the n8n **cost** or the **fragility** of the workflows? (Cost alone → self-host n8n may be enough. Fragility → move to code-first.)
- Comfort level writing TypeScript, directly or with an AI assistant.
- A per-source inventory: which of the 20–40 systems Airbyte already covers.

## 9. Open threads / next questions

- A shared connector pattern so the custom connectors don't become twenty one-offs (common interface: auth, incremental cursor, pagination, raw landing table). Started in section 8.
- Real-time versus good-enough tradeoffs, source by source.
- The matching logic in depth — thresholds, and a review queue for uncertain matches.
- Moving from visualization to **action** from the dashboard — and how to do it without welding the insight and action planes together.
