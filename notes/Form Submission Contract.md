---
tags: [gtm-os, build, contract, standard]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [webhook payload, CloudEvents, Standard Webhooks]
---

# Form Submission Contract

Because both ends are ours (the WordPress plugin *and* the receiver), we **design the contract** instead of accepting whatever the form emits.

> [!warning] Not yet verified
> The payloads below came from another chat and are **illustrative** (`example.com`, a documentation IP range `203.0.113.x`). Before building on them, fire the plugin's **Send test event** at the receiver and capture the real body **and headers**.

## Payload (v0, as described)
```json
{
  "event_id": "uuid",
  "form": "contact | subscribe | test",
  "submitted_at": "ISO-8601",
  "site": "https://galliot.us/",
  "fields": { "name": "…", "email": "…", "message": "…" },
  "context": {
    "ip": "…", "ip_hash": "…", "country": "US",
    "user_agent": "…", "browser": "…", "os": "…", "device": "desktop",
    "automated_user_agent": false, "accept_language": "…", "language": "…", "timezone": "…",
    "page_url": "…", "page_title": "…", "referrer": "…", "utm": { "utm_source": "…" },
    "seconds_on_form": 18.4, "captcha": "passed",
    "email_domain": "…", "email_free_provider": false, "email_disposable": false, "email_domain_has_mx": true,
    "message_length": 51, "message_link_count": 0, "message_non_latin_percent": 0
  }
}
```
Subscribe form: `"fields": { "email": "…" }`. Test event: `form: "test"`, minimal context.

## What's right
- **Envelope pattern**: metadata wraps the payload (`fields`) and its `context`.
- **`form` as a discriminator**: tells the receiver how to read `fields`.
- **`event_id`** = the **idempotency key**, stored as the **external ID** (the core still mints its own [[Correlation ID]]).
- **Spam signals at the source**: good **feature extraction**. The decision stays in the core.

## What to fix for v1
- [ ] **Add a schema version** (`specversion` / `version`). Without it, the first field rename breaks old events.
- [ ] **Drop raw `ip`; keep `ip_hash`** unless there's a concrete need. That's **data minimisation**.
- [ ] *(Nice to have)* split `context` (facts about the visitor) from `signals` (derived spam features).
- [ ] **Test events never become business events**: log as a technical check, then drop.

## Standards to adopt
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

### Delivery
**Standard Webhooks** standardises how webhooks are sent and verified: headers `webhook-id`, `webhook-timestamp` and `webhook-signature` (**HMAC** with a shared secret), with official verification libraries. The timestamp blocks **replay attacks** (an old signed request re-sent).

**Recommendation:** v1 = this payload in a CloudEvents envelope, delivered with Standard Webhooks headers. Small plugin change; no home-made format.

## Questions for the plugin code
- ~~Does it sign requests today?~~ **Yes**: `X-Galliot-Signature`. Not yet verified by the receiver.
- Does it retry on failure? How often, and does it reuse the same `event_id`?

Used by [[Form Intake Pipeline]].
