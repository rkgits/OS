---
tags: [gtm-os, build, ticket]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [FORM-INTAKE-v0]
---

# Ticket: Form Intake v0 — website form → receiver → Trigger.dev → spam decision

> [!info] For a fresh Claude Code session
> This ticket is self-contained. Background lives in the Obsidian vault in this repo (`rkgits/OS`, folder `notes/`); start at [[GTM OS - Map of Content]]. The notes most relevant here are [[Form Intake Pipeline]], [[Form Submission Contract]], [[Engine vs Core]], [[Correlation ID]], [[Telemetry Module]] and [[Two Log Streams]]. Read those six before writing code.

## Why
Reza (GTM engineer at Galliot) is replacing a pile of organically grown n8n workflows with a deliberately designed "GTM OS". n8n's per-execution pricing and the workflows' fragility are the two problems. This ticket is the **first real build** and the template every later worker will copy. So **patterns matter more than features**: idempotency, correlation IDs, structured logging, and logic owned by our code instead of the orchestrator.

Today, form submissions on **galliot.us** (custom WordPress plugin "Galliot Forms") are emailed to an inbox. The plugin already supports posting to a webhook URL (*Settings > Galliot Forms*), but none is configured.

## Goal
When someone submits a form on galliot.us, the submission is received, stored raw, traced with a correlation ID, and classified as **spam / accepted / needs review**, with every step visible in logs and as business events.

## Architecture
```
WordPress (Galliot Forms)
   │  POST JSON (webhook)
   ▼
Receiver — Supabase Edge Function          ← core: verify, dedupe, mint correlation ID, store raw
   │  trigger task (idempotency key = source event_id)
   ▼
Trigger.dev task  form-submission-received   ← thin wrapper
   │  calls
   ▼
core/spam.ts  classifySubmission()           ← pure function, engine-agnostic
   │
   ▼
Supabase: form_submissions (raw + decision) · business_events
```

**Why a receiver at all:** Trigger.dev tasks are started from your own code; its webhook guides all put an endpoint you own in front ([webhook guides](https://trigger.dev/docs/guides/frameworks/webhooks-guides-overview), including a Supabase Edge Function example). Calling Trigger.dev's API straight from WordPress would mean putting the Trigger.dev **secret key** in the plugin. **Not allowed.**

## Non-negotiable principles
1. **Engine vs core.** Business logic lives in plain TypeScript functions under `src/core/` with no Trigger.dev imports. Trigger.dev tasks are thin wrappers (a few lines) that call core functions. Test: swapping Trigger.dev for Inngest should only touch `src/tasks/`.
2. **Correlation ID minted by our code**, in the receiver, on first contact. Passed into the task and onto every log line and business event. Trigger.dev's run ID is recorded as an *extra* field, never as the primary trace ID.
3. **Idempotency.** The source `event_id` (a UUID minted by WordPress) is the external ID: unique in the database, and used as the Trigger.dev idempotency key. A duplicate delivery is acknowledged and not processed again.
4. **Store raw first, untouched**, before any processing. Bad parsing must never lose a submission.
5. **Two log streams.** Technical logs (for maintainers, noisy, short retention) vs **business events** (stable names, a Supabase table, read by dashboards). See [[Two Log Streams]].
6. **No personal data in technical logs.** IP, user agent, email, name and message text go only in the raw table. Logs carry IDs, statuses, durations and error messages.
7. **Respond fast.** The receiver answers WordPress as soon as the row is stored and the task is triggered; it never waits for classification.

## Scope

### In scope
- Supabase tables and migration (see *Data model*).
- Receiver Edge Function.
- Minimal **telemetry module** (`src/core/telemetry.ts`).
- One Trigger.dev task: `form-submission-received`.
- Rule-based spam classifier with two thresholds → `spam` / `accepted` / `needs_review`.
- Business events for the lifecycle.
- Configure the Galliot Forms webhook URL and verify end to end with **Send test event** plus one real submission.
- A short README: setup, env vars, how to run locally, how to deploy.

### Out of scope (later tickets; do not build)
- **Any change to the WordPress plugin code** (Reza's explicit instruction). That includes HMAC signing and the CloudEvents envelope; see *Known gaps*.
- The LLM judge for the grey zone, review-queue UI, notifications to Slack/CRM, enrichment (domain/LinkedIn), dashboards.
- OpenTelemetry export to an external backend.

## Inputs: what WordPress sends (unverified)
The shape below was described in another chat and is **illustrative**. **Step 1 of this ticket is to capture the real body and headers** and adjust.

```json
{
  "event_id": "uuid",
  "form": "contact | subscribe | test",
  "submitted_at": "2026-10-12T21:30:15+00:00",
  "site": "https://galliot.us/",
  "fields": { "name": "…", "email": "…", "message": "…" },
  "context": {
    "ip": "…", "ip_hash": "…", "country": "US",
    "user_agent": "…", "browser": "Chrome 130", "os": "macOS", "device": "desktop",
    "automated_user_agent": false, "language": "en-US", "timezone": "…",
    "page_url": "…", "page_title": "…", "referrer": "…", "utm": {},
    "seconds_on_form": 18.4, "captcha": "passed",
    "email_domain": "…", "email_free_provider": false, "email_disposable": false, "email_domain_has_mx": true,
    "message_length": 51, "message_link_count": 0, "message_non_latin_percent": 0
  }
}
```
- `subscribe` sends only `fields.email`. `test` is sent by the **Send test event** button: `fields.message`, minimal context, `ip_hash: "test"`.
- Requests are **believed to be unsigned**. Whether the plugin retries on failure (and with the same `event_id`) is unknown.

## Data model (Supabase / Postgres)

**`form_submissions`**: one row per delivery, raw plus decision
| column | type | notes |
| --- | --- | --- |
| `id` | uuid pk | |
| `correlation_id` | uuid not null | minted by the receiver |
| `source_event_id` | text **unique** | WordPress `event_id`; the idempotency key |
| `source` | text | e.g. `galliot.us` |
| `form` | text | `contact` / `subscribe` / `test` |
| `is_test` | bool | `form = 'test'` |
| `submitted_at` | timestamptz | from the payload |
| `received_at` | timestamptz default now() | |
| `raw_body` | jsonb | exactly as received |
| `raw_headers` | jsonb | as received, minus our auth token |
| `status` | text | `received` → `processing` → `classified` / `failed` |
| `decision` | text null | `spam` / `accepted` / `needs_review` |
| `spam_score` | numeric null | |
| `decision_reasons` | jsonb null | which rules fired |
| `rules_version` | text null | e.g. `rules-v0` |
| `engine_run_id` | text null | Trigger.dev run ID |
| `classified_at` | timestamptz null | |

**`business_events`**: the event log (the backbone; dashboards read this)
| column | type | notes |
| --- | --- | --- |
| `id` | uuid pk | |
| `correlation_id` | uuid not null | indexed |
| `event_type` | text not null | e.g. `form_submission.received` |
| `event_version` | int default 1 | |
| `subject_type` | text | `form_submission` |
| `subject_id` | uuid | → `form_submissions.id` |
| `occurred_at` | timestamptz default now() | |
| `actor` | text | worker name, e.g. `receiver`, `spam-classifier` |
| `data` | jsonb | small, **no personal data** (decision, score, form, reasons) |

Event types for v0: `form_submission.received`, `form_submission.classified` (`data.decision` = spam / accepted / needs_review), `form_submission.failed`. **Test events produce no business events.**

Enable RLS on both tables; only the service role writes.

## Components

### 1. Receiver (Supabase Edge Function `form-intake`)
1. Accept `POST` only; reject a body over ~64 KB.
2. **Interim auth:** require a secret token, sent as `?token=` in the webhook URL (or a custom header, if the plugin supports one without code changes). Compare in constant time. Reject with `401` and log a warning. *This is weaker than HMAC signing and is accepted as a temporary gap; see Known gaps.*
3. Parse JSON. On parse failure, still store the raw text with `status = failed`, return `400`, and log an error.
4. Validate the minimal envelope (`event_id`, `form`, `submitted_at`) with a schema (e.g. Zod). Unknown extra fields are allowed and kept.
5. **Idempotency:** `INSERT … ON CONFLICT (source_event_id) DO NOTHING`. If it already existed, return `200 {"status":"duplicate"}` and don't trigger.
6. Mint `correlation_id`, store the row (`status = received`), and write `form_submission.received` (skip for test events).
7. Trigger `form-submission-received` with payload `{ submissionId, correlationId }` (IDs only, no personal data) and `idempotencyKey = source_event_id`. Follow Trigger.dev's Supabase Edge Function guide for how to trigger from Deno.
8. Return `202 {"status":"accepted","correlation_id":…}`.
9. If the trigger call fails: keep the row as `received`, log an error, **still return 202** (the submission is safe; reprocessing is a later ticket). Note this in the README.

### 2. Telemetry module (`src/core/telemetry.ts`), about 100 lines
Spec from [[Telemetry Module]]:
- `withContext({ correlationId?, worker, runId?, subjectId? })` → a logger. Reuses an incoming correlation ID or mints one.
- `log.info / log.warn / log.error(message, fields?)` → **structured JSON** lines with standard fields: `time, level, worker, correlation_id, run_id, subject_id, action, status, duration_ms, error`.
- `log.event(eventType, data)` → inserts into `business_events`.
- **Redaction:** drop or mask keys like `token`, `secret`, `authorization`, `api_key`, `email`, `ip`, `user_agent`, `message`, `name`.
- **Never throws.** A logging failure is swallowed (and printed with `console.error` at most); the job always continues.
- Build on a standard logger if convenient (Pino); keep the field names stable.
- Shared by the receiver and the task. If sharing code between Deno (Edge Function) and Node (Trigger.dev) is awkward, keep the module dependency-free TypeScript that runs in both.

### 3. Trigger.dev task `form-submission-received` (thin wrapper)
- Load the submission by ID, set `status = processing`, call `classifySubmission(rawBody)`, persist the decision fields and `engine_run_id`, write `form_submission.classified`, set `status = classified`.
- Test events: classify (useful as a smoke test) but write no business event.
- Retries: use Trigger.dev's built-in retry (e.g. 3 attempts with backoff). On final failure: `status = failed` + `form_submission.failed`.
- Use the current Trigger.dev SDK (v4 at time of writing); **check the docs** rather than assuming APIs.

### 4. Spam classifier (`src/core/spam.ts`, a pure function)
`classifySubmission(payload) → { decision, score, reasons[], rulesVersion }`. No I/O. Signals are already computed by the plugin; **the decision is made here**, so rules change without touching WordPress.

Starting rules (`rules-v0`; weights live in one config object, easy to tune):
| Signal | Effect |
| --- | --- |
| `captcha != "passed"` | strong spam |
| `automated_user_agent = true` | strong spam |
| `email_disposable = true` | strong spam |
| `email_domain_has_mx = false` | strong spam |
| `seconds_on_form < 3` | strong spam |
| `message_link_count >= 2` | moderate spam |
| contact form and `message_length < 10` | moderate spam |
| `email_free_provider = true` on the contact form | weak spam (B2B consulting site) |
| `message_non_latin_percent > 50` | weak spam |
| very old browser major version (e.g. Chrome < 115) | weak spam |

**Two thresholds:** score ≥ `HIGH` → `spam`; score ≤ `LOW` → `accepted`; in between → `needs_review` (stored only; no review UI yet). Record every rule that fired in `reasons`.

**Golden test case** (a real submission, Oct 18 2026): message "Hy" (2 chars), sender `…seoexpert@gmail.com` (free provider), Chrome 109 on Windows, from PK, 25.4 s on form, **captcha passed**, 0 links. Expected: **`spam`**, or at minimum `needs_review`. Captcha alone must not save it. Also add fixtures for a clearly legitimate business inquiry (→ `accepted`) and for a `subscribe` with a free email (→ `accepted`; the free-provider rule applies to contact only).

## Steps
1. **Setup.** Confirm with Reza where the code lives: a new repository (suggested `gtm-os-workers`), *not* the `OS` notes vault. Install the Trigger.dev CLI and MCP (`npx trigger.dev@latest install-mcp`, then log in). Reza has a **free Trigger.dev account**. Confirm which Supabase project to use and install the Supabase CLI.
2. **Capture the real payload.** Deploy a bare receiver that only stores body + headers and returns 200. Reza points Galliot Forms at it and presses **Send test event**. Compare against *Inputs* above, update [[Form Submission Contract]] with what really arrives (body, headers, signed or not), and remove the "unverified" warning.
3. **Migration** for both tables + RLS.
4. **Telemetry module** + unit tests (redaction; never throws).
5. **Spam classifier** + unit tests from the fixtures above.
6. **Full receiver**: auth token, validation, idempotency, correlation ID, trigger.
7. **Trigger.dev task**, deployed to the dev environment.
8. **End-to-end tests**: test event; a real contact submission; the **same payload posted twice** (one row, one run); a bad token (401); malformed JSON (400 + stored as failed).
9. **Go live**: set the production webhook URL in Galliot Forms. Leave email delivery on if the plugin allows both, and run in parallel for a few days to compare decisions against the inbox.
10. **Update the vault**: mark this ticket done, record what was learned in [[Form Intake Pipeline]] and [[Form Submission Contract]], and update [[Open Threads]] (per this repo's `CLAUDE.md`).

## Acceptance criteria
- [ ] A real form submission on galliot.us appears in `form_submissions` within seconds, with `raw_body` and `raw_headers` intact.
- [ ] Every submission gets exactly one `correlation_id`, present on its row, its business events and every technical log line, and searchable in the Trigger.dev dashboard.
- [ ] Posting the same payload twice results in one row and one task run.
- [ ] Requests without the correct token are rejected (401) and never reach the database.
- [ ] The golden spam case is classified `spam` (or `needs_review`) with reasons recorded; the legitimate fixture is `accepted`.
- [ ] Test events are stored and classified but produce no business events.
- [ ] No IP, email, name, user agent or message text appears in technical logs or `business_events.data`.
- [ ] `src/core/` has no Trigger.dev imports; tasks are thin wrappers.
- [ ] No secrets in code or in WordPress except the receiver token; the Trigger.dev secret key exists only in Supabase function secrets.
- [ ] README covers setup, env vars, local run, deploy, and the known gaps.

## Known gaps (accepted for v0, tracked for v1)
- **No request signing.** The URL token is an interim measure. v1: the plugin signs with **Standard Webhooks** headers (`webhook-id`, `webhook-timestamp`, `webhook-signature`, HMAC), which also blocks replay attacks.
- **No schema version, and not a CloudEvents envelope.** v1: wrap the payload in **CloudEvents** (`specversion`, `id`, `source`, `type` such as `us.galliot.form.contact.submitted`, `time`, `data`). See [[Form Submission Contract#Standards to adopt]].
- **Raw IP is sent alongside `ip_hash`** (data minimisation). v1: send the hash only.
- **No reprocessing** of rows stuck in `received` if triggering fails.
- **`needs_review` has no queue UI** and no LLM judge yet.

## Glossary (terms used above)
**Idempotency**: processing the same input twice has the same effect as once. **Correlation ID**: one ID following an item through every step. **Business event**: a stable, human-meaningful record of what happened ("classified as spam"), as opposed to a technical log line. **Thin wrapper**: orchestrator code that only calls core functions. **RLS**: Postgres row-level security.
