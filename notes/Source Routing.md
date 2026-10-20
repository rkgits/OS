---
tags: [gtm-os, ingestion, architecture, decision]
status: seed
created: 2026-10-19
updated: 2026-10-19
aliases: [source attribution, per-source config, routing]
---

# Source Routing

**One front door, one inbox, one event log, then a different workflow per source.** "Same front door, different rooms."

## The case
A second site, **Life Science Tech Collective** (`lifesciencetechcollective.com`, repo `galliot-us/LSTC`), has a signup form. Its submissions should flow through the same [[Webhook Ingress|receiver]] as galliot.us, but get a different treatment: no spam classification; instead **subscriber onboarding** (welcome email, newsletter list).

## Corrections
- These are **form submissions**, not emails. Email was only how they used to be delivered.
- "Tag them differently" has a name: **source attribution**.
- "A similar format" isn't good enough. Aim for a **shared contract** ([[Form Submission Contract]]); "similar" is what breaks parsers. New code sends **v1** from day one.
- What you're describing is **routing**: a **dispatcher** picks the workflow by source.

## Rules
1. **Never trust the payload to say where it came from.** The receiver derives the source from *how the request authenticated* (which credential or route), not from a `site` field in the body. *(GTM-Engine v0 currently reads `body.site`; fixed in [GTM-Engine #2](https://github.com/galliot-us/GTM-Engine/issues/2).)*
2. **Per-source credentials**: one token per server-side source, so each can be revoked alone.
3. **Per-source configuration**: auth, allowed origins, workflow and spam rules are config entries, not `if` statements spread through the code. Galliot's "free Gmail address" penalty would flag most genuine community members as spam.
4. **One event log, filtered by source.** Visibility comes from `business_events` with a `source_id`, not separate pipelines.
5. **Idempotency keys are per source**: `unique (source_id, source_event_id)`.

## Trust: server sources vs public sources
> [!warning] Correction (including an earlier recommendation)
> "Give each site its own secret token" and "sign the v1 contract" **don't work for LSTC as built**. It's a static site and its form posts **from the visitor's browser**. **A browser can't keep a secret**: anything shipped to it is public.

| | Server source (galliot.us WordPress) | Public source (LSTC static site) |
| --- | --- | --- |
| Who sends | Your server | A visitor's browser |
| Can hold a secret | Yes | **No** |
| Proves the sender with | Token, then HMAC signature | Nothing cryptographic |
| Protections | Signature + timestamp (replay) | **Origin allowlist** (stops other websites, not bots), **rate limits**, honeypot, later **Turnstile** |

Even a server token only proves "this came through my server", **not** "a real person typed this". **Authenticating the source ≠ verifying the submitter.**

## Side effects
A welcome email is a **side effect**: an action in the outside world you can't take back.
- Make it **idempotent**: check for a `welcome_sent` business event before sending; write it right after.
- On a public endpoint, anyone can submit *someone else's* address and make you email them (**subscription bombing**). Rate limits contain it; **double opt-in** (confirm link) fixes it.
- **Defer, don't skip.** Deferring a transactional email service (Postmark, Resend) and Substack automation is fine. Skipping the cheap safeguards (idempotency check, rate limit) isn't.

## Substack
Verified (Oct 2026): **no official subscriber API**. The only programmatic route is an undocumented endpoint, reportedly returning 403 since late 2025 ([laserfocus](https://laserfocus.substack.com/p/automate-adding-substack-subscribers), [smtpedia](https://smtpedia.com/platforms/substack/api/)). Options: CSV import, or a newsletter platform with a documented API (beehiiv, which Galliot already uses). Don't design around Substack automation.

## Work
- [GTM-Engine #2](https://github.com/galliot-us/GTM-Engine/issues/2): per-source credentials, source attribution, public endpoint
- [LSTC #1](https://github.com/galliot-us/LSTC/issues/1): send signups in the shared v1 contract
- [GTM-Engine #3](https://github.com/galliot-us/GTM-Engine/issues/3): dispatcher + LSTC onboarding workflow

Related: [[Worker Coordination]], [[Engine vs Core]], [[Two Log Streams]].
