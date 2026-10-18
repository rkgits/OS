---
tags: [gtm-os, foundations]
status: evergreen
created: 2026-10-15
updated: 2026-10-18
---

# Data Integration

The overarching discipline is **data integration**. The thing being built is a **data pipeline**: extract data from many sources, land it somewhere central, transform it into clean reporting tables, and visualize it.

Two classic pipeline patterns:

- **ETL**: Extract, Transform, Load. Clean and reshape the data *before* it lands.
- **ELT**: Extract, Load, Transform. Land raw data first, transform later with SQL. The modern warehouse approach, and the one to use here.

Where the data lands: [[Integration Approaches]] → [[Choosing the Store]].
How it gets there: [[Ingestion Models]].

Back to [[GTM OS - Map of Content]].
