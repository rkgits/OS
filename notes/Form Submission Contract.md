---
tags: [gtm-os, build, contract, standard]
status: growing
created: 2026-10-19
updated: 2026-10-21
aliases: [webhook payload, CloudEvents, Standard Webhooks]
---

# Form Submission Contract

Because both ends are ours (the WordPress plugin *and* the receiver), we can shape the contract instead of accepting whatever the form emits. Everything below was **verified against real deliveries** on Oct 19, 2026, plus the plugin's source (`galliot-us/galliot-website`, `docs/forms.md` and `inc/forms.php`).

> [!info] Verified, not illustrative
> The first version of this note used a payload described in another chat. A real **Send test webhook** delivery and a real contact submission from galliot.us have since been captured. The shape was close, but the real traffic exposed quirks (see [[#Quirks the real traffic exposed]]).

## What really arrives

**A test event** (the plugin's **Send test webhook** button), body exactly as received:
```json
{
  "event_id": "7c0e7006-…",
  "form": "test",
  "submitted_at": "2026-10-19T20:16:17+00:00",
  "site": "https://galliot.us/",
  "fields": { "message": "Test event from Settings > Galliot Forms" },
  "context": { "page_url": "https://galliot.us/wp-admin/options-general.php?page=galliot-forms", "ip_hash": "test" }
}
```

**A contact submission** has the same envelope with `form: "contact"`, `fields` = `name`, `email`, `message`, and a full `context` (the plugin's own list, with the spam signals): `ip`, `ip_hash`, `country`, `user_agent`, `browser`, `os`, `device`, `automated_user_agent`, `accept_language`, `language`, `timezone`, `page_url`, `page_title`, `referrer`, `utm`, `seconds_on_form`, `captcha`, `email_domain`, `email_free_provider`, `email_disposable`, `email_domain_has_mx`, `message_length`, `message_link_count`, `message_non_latin_percent`. A subscribe form sends only `fields.email` and no `message_*` values.

**Headers** that matter: `Content-Type: application/json`, `User-Agent: GalliotForms/1.0`, and `X-Galliot-Timestamp` (unix seconds). When a signing secret is configured it also sends `X-Galliot-Signature`.

## How the plugin delivers
```
visitor → browser script → POST /wp-json/galliot/v1/form
   → honeypot, minimum time (1.5 s), rate limit (5 per 10 min per IP), validation, reCAPTCHA
   → signed JSON POST to the webhook
        success: done
        failure: queued, retried at 5 min, 15 min, 1 h, 3 h, 6 h
        still failing: one fallback email, then the item is dropped
```
- WordPress **stores no entries**. The only copy is a queue item while a delivery is failing. The receiver's database is the record.
- **No webhook configured** → the plugin emails the fallback address instead. Setting a webhook URL therefore *replaces* email delivery; this matters when switching on production.
- A retry after a timeout can deliver the **same `event_id` twice**, so the receiver must deduplicate ([[Webhook Ingress]]).

## Plugin signing: it exists, and it's its own scheme
> [!warning] Correction
> The earlier assumption "requests are unsigned" was wrong. The plugin can sign. The **signing secret is optional**: with no secret set, the signature header is simply absent.

- **Header:** `X-Galliot-Signature: sha256=<hex>`.
- **What is signed:** HMAC-SHA256 of `"<timestamp>.<raw body>"` with the shared secret, where the timestamp is the `X-Galliot-Timestamp` value.
- **Verification** means recomputing that HMAC over the *raw* body (before any JSON parsing) and rejecting timestamps older than about five minutes (replay protection).
- This is **not Standard Webhooks** (different header names, no `webhook-id`). It's close in spirit: a timestamp in the signed content plus an HMAC.
- **Observed so far:** only unsigned deliveries (no secret was set). The signed form comes from the plugin source and has not been seen live.
- **v0 state:** the receiver checks a secret **URL token** and does *not* verify the signature. A URL token is weaker: it can leak into server logs, and it gives no replay protection.

## Quirks the real traffic exposed
> [!warning] Be a tolerant reader
> The first real submission **failed validation** and only the stored raw row and the failure path saved it. The contract must accept what the sender really produces.

- **PHP encodes an empty array as `[]`, not `{}`.** An empty `utm` arrived as `[]`; a filled one is an object. A schema that says "object" rejects every submission without campaign tags.
- **Any `context` value can be `null`.** The plugin documents that `country` is `null` off Cloudflare and `email_domain_has_mx` is `null` when DNS can't be checked.
- **Test events have a minimal context** (`page_url`, `ip_hash: "test"`): no IP, no user agent.
- **Lesson (the robustness principle):** be strict about what you *send* and tolerant about what you *accept*; treat every `context` field as optional and nullable, because a missing signal must never reject a submission. The raw copy is stored **before** parsing for exactly this reason ([[Form Intake Pipeline]]).

## What's right
- **Envelope pattern**: metadata wraps the payload (`fields`) and its `context`.
- **`form` as a discriminator**: tells the receiver how to read `fields`.
- **`event_id`** = the **idempotency key**, stored as the **external ID** (the core still mints its own [[Correlation ID]]).
- **Spam signals at the source**: good **feature extraction**. The decision stays in the core.
- **Test events never become business events**: stored and classified as a smoke test, no event written ([[Two Log Streams]]).

## Design points for v1
- **Add a schema version** (`specversion` / `version`). Without it, the first field rename breaks old events.
- **Drop raw `ip`; keep `ip_hash`** unless there's a concrete need. That's **data minimisation**; the plugin docs already flag the raw IP as personal data.
- *(Nice to have)* split `context` (facts about the visitor) from `signals` (derived spam features).
- **Which signing scheme to standardise on** is an open design question: see below.

## Standards
### Envelope: CloudEvents
**CloudEvents** (CNCF, the same foundation as [[OpenTelemetry]]) standardises event metadata. Mapping:

| Ours | CloudEvents |
| --- | --- |
| `event_id` | `id` |
| `site` | `source` |
| `form` | `type`, e.g. `us.galliot.form.contact.submitted` |
| `submitted_at` | `time` |
| `fields` + `context` | `data` |
| *(new)* | `specversion: "1.0"`, `datacontenttype: "application/json"` |

### Delivery: Standard Webhooks vs the plugin's own scheme
**Standard Webhooks** standardises how webhooks are sent and verified: headers `webhook-id`, `webhook-timestamp` and `webhook-signature` (**HMAC** with a shared secret), with official verification libraries. The timestamp blocks **replay attacks**.

The trade-off, now that we know the plugin already signs:
- **Verify the existing scheme:** no plugin change; a few lines of HMAC in the receiver. Fine for one site.
- **Move to Standard Webhooks:** needs a plugin change, but it buys library support and a *shared* delivery convention for every future source ([[Source Routing]]).

No decision yet; the question is tracked in [[Open Threads]].

Used by [[Form Intake Pipeline]].
