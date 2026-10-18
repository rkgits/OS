---
tags: [gtm-os, foundations, decision]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
---

# Choosing the Store

**Decision: start on Supabase (hosted Postgres).** At 20–40 GTM systems the volume is millions of rows, not billions. Move to Snowflake only when aggregations start to crawl.

| Store | What it really is | Good for | Ceiling |
| --- | --- | --- | --- |
| Airtable | Spreadsheet-database hybrid | Small, human-managed data | Tens of thousands of rows; not an analytical backend |
| Supabase | Hosted **Postgres**, relational and transactional (**OLTP**) | Cheap, developer-friendly backend | Millions of rows, comfortably |
| Snowflake | Analytical warehouse (**OLAP**) | Aggregating billions of rows fast | Scale not needed yet |

Why Postgres is right *for the right reason*: [[OLTP vs OLAP#The key mental correction]].
