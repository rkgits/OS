---
tags: [gtm-os, identity, data-model]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
---

# Person vs Account

A two-layer, **one-to-many** hierarchy:
- **Person** (bottom): every actual touch happens here; you can't email a company.
- **Account** (top): one company, many people. Revenue closes here, so reporting leans here.

**Resolve identity at the person level first, then roll up to accounts.** Account metrics are aggregations of person-level events. "Accounts matter, not people" is a dangerous half-truth.

**People change companies.** The person→account link must change over time without erasing history. That is a **slowly changing dimension**; ignore it and historical account numbers quietly rot.

Built on [[Identity Resolution]]. Part of the shared data model in [[System Design Principles]].
