---
tags: [gtm-os, security, ingestion, concept]
status: seed
created: 2026-10-28
updated: 2026-10-28
aliases: [honeypot, defence in depth, spam protection]
---

# Bot Protection

## Honeypot
A **honeypot** is a trap for bots: a form field that **real people never see** but **automated scripts fill in**, because they fill every field they find. If it comes back filled, a bot almost certainly sent the form. (The name comes from security work, where a honeypot is any decoy built to attract attackers.)

**Example:** the LSTC signup form has a field called `website`, moved off-screen, skipped by the Tab key and hidden from screen readers. A human never touches it.

> [!warning] Corrections
> 1. **It only catches dumb bots.** Generic scripts fall for it; a bot written for your form, or one driving a real browser, leaves the field empty. It's a **cheap filter, not real protection.**
> 2. **A honeypot checked in the browser does almost nothing.** A bot posting straight to your endpoint never runs your page's JavaScript. **It only counts when the server checks it.** That's why the LSTC form should *send* `honeypot_filled` and the receiver should reject it ([[Source Routing]]).

## Defence in depth
No single check is enough, so you **layer cheap checks**, each catching what the others miss:

honeypot → minimum time on form (bots submit almost instantly) → **rate limits** → a CAPTCHA (e.g. **Turnstile**, Cloudflare's CAPTCHA alternative) → rule-based scoring in the core ([[Form Intake Pipeline#The spam decision]]).

The galliot.us plugin already does several of these (honeypot, minimum time, rate limit, reCAPTCHA), and the real spam case still passed the captcha. That's the argument for layers.

Related: [[Source Routing#Trust: server sources vs public sources]].
