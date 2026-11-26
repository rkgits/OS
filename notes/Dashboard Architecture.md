---
tags: [gtm-os, architecture, interface, security, decision]
status: growing
created: 2026-11-25
updated: 2026-11-25
aliases: [dashboard, interface layer, layer 5, authentication vs authorisation, definer view]
---

# Dashboard Architecture

**Layer 5 of the [[System Map]]: the interface.** What the first dashboard is, the rules it follows, and why. Built and live (GTM-Engine [epic #4](https://github.com/galliot-us/GTM-Engine/issues/4)); this note holds the reasoning, the issues hold the work.

## The shape
```
Browser (a static app on AWS Amplify) ──sign in──▶ Supabase Auth
        │  a signed login token (JWT)
        └──────────▶ Postgres: ONE view, dashboard_inbox
                       rows come back only if the caller is on an allowlist
                       the real tables are closed to the browser entirely

The engine (receiver → worker) writes the tables with a powerful key that never leaves the server.
```
There is **no server of our own** behind the dashboard. The browser talks to the database directly, so **the database is the security boundary**, not the app.

## Rules (decisions marked)
1. **The dashboard reads only from the core store, never from vendor APIs.** That is the old trap again ([[Integration Approaches]], [[Ingestion Layer Options]]): the dashboard would be welded to every vendor's limits and have no history.
2. **Decision: it never reads the tables themselves, only one purpose-built view.** The view lists exactly the fields the inbox needs, normalised across sources and contract versions, and nothing else ([[Form Submission Contract]]). A new column in a table can never leak into the browser by accident, because the view has to be changed on purpose.
3. **It is read-only.** Showing is the **system of insight**; acting is a separate **system of action** ([[Ingestion Models]]). Don't weld them together.

## Correction: logging in is not permission to read
> [!warning] Two things that look like one
> **Authentication** answers *who are you?* **Authorisation** answers *what may you read?* A login page does the first. It says nothing about the second.

Here they are two separate locks that fail independently:
- **Authentication:** accounts are invite-only (public sign-ups, anonymous sign-ins and magic links are all switched off), so strangers cannot even get an identity.
- **Authorisation:** an **allowlist** table. Even a perfectly valid account sees **zero rows** unless it is on the list. This was tested with a throwaway "stranger" account: signed in, confirmed, and shown nothing.

If only the first lock existed, anyone who ever got an account (a leaked invite, a mistake in settings) would see everything. Same logic as [[Source Routing#Trust: server sources vs public sources]]: proving who someone is is a different question from what they may do.

## The key that is not a secret
The page ships to the browser, so **everything in it is public**: anyone can open the code and read it. Two keys show why this matters:
- The **anon key** is *designed* to be public. It only says "this request is for our project, as an anonymous visitor". Its power is whatever the database grants to anonymous visitors, and we granted **nothing**.
- The **service-role key** bypasses every rule. It lives only on servers. If it ever reached the browser bundle, the whole database would be open.

**The lesson: you can't protect data by hiding a key in a public page; the protection has to live in the database.** CI scans every build for the dangerous key, and fails the build if it appears.

## Pattern: a definer view plus an allowlist
A **view** is a saved query. In Postgres a plain view runs with its **owner's** rights by default (a **definer** view); you can opt in to the *caller's* rights instead (**security invoker**). We use the default **on purpose**: the owner can read the real tables, the logged-in user cannot, and that is how the browser gets some fields without getting the table. (The earlier owner-only view of LSTC signups is the opposite: invoker, so it respects the table's rules.)

> [!warning] That makes it dangerous unless guarded
> A definer view is a controlled hole in a wall. It is safe only because of three guards:
> 1. **The allowlist check runs inside the view, on every request:** rows come back only if the caller is on the list.
> 2. **A security barrier:** a caller's own filters can't run before that check and be used to probe hidden rows.
> 3. **Grants:** nothing for the public roles; read-only for logged-in users.

**Defence in depth** ([[Bot Protection#Defence in depth]]): the real tables are *also* closed with row-level security **and** by removing the public roles' privileges. Supabase hands every new table default privileges to the public roles, which means row-level security alone was the only guard. One forgotten policy would have exposed raw submissions. Removing the privileges means two independent things must both fail.

## Why removing someone is immediate
A login token says **who you are**, not **what you may read**. The database checks the allowlist on every single request. So taking someone off the list cuts their data access **instantly**, even if they hold a token issued five minutes ago. Systems that put permissions *inside* the token have the opposite problem: the permission stays valid until the token expires. We also block new sign-ins, so the removed account can't come back with a password. Both effects are tested.

## Choices
- **Supabase Auth, not AWS Cognito.** The data and its access rules already live in Supabase, so Postgres can check the logged-in user directly. Cognito would be a second identity system, plus a bridge so Supabase trusts its tokens, for no gain at this size. *(Revisit only if the company already standardises on Cognito or needs SSO.)*
- **AWS Amplify as hosting only.** It serves static files. We don't use its login, database or functions, so leaving Amplify later means moving a folder of files, not rewriting an app ([[Engine vs Core]] again, applied to hosting).
- **One environment.** An internal tool doesn't earn two. A second would be a second hosting branch plus its own database project, and nothing in the code is environment-specific: it is all settings.
- **Two-step verification (TOTP) is optional for now.** The app can enrol it. Making it **mandatory** is a trade-off: it needs a check inside the view, and until someone enrols it locks them out. Open question below.
- **CI is a signal, not a gate.** The paid plan needed to *block* merges on a failing check isn't enabled, so the discipline is ours ([[Engineering Basics]]).

## Proving it: a safeguard you've never seen fail is only a belief
Every guard above was checked by a script that tries to break in as three callers: **no login**, **logged in but not on the list**, **on the list**. Then the checks were themselves broken on purpose (grant the public role access; ignore the filter; show stale rows) to prove they turn red. The method found real problems that reading the code did not:
- The hosting default rewrite rule made deep links answer with a redirect and an error status. The hosting check caught it.
- The engine's test runner started picking up the dashboard's tests and would have failed CI. Caught by running the full check once more before opening the change.
- My own patch to a check script silently did nothing because its anchor text didn't match. Caught because the expected new lines never appeared in the output.

> [!tip] Habit
> **Prove a safeguard by breaking it.** A passing check on its own tells you little; a check you've watched fail, then pass after the fix, tells you something.

## What stays out of v0
Charts, search, date ranges, export, editing or deleting data, mandatory two-step verification, a second environment, roles beyond "viewer", and showing the **full** message (the view carries a 200-character preview on purpose, so exposing more is a decision, not a default).

## Open
- **Mandatory two-step verification?** See [[Open Threads]].
- **How long do we keep the personal data this shows?** [[Personal Data and Retention]].

Related: [[Webhook Ingress]] (the other front door), [[Two Log Streams]] (why the dashboard's numbers will come from business events), [[Recurring Lessons]]. Back to [[GTM OS - Map of Content]].
