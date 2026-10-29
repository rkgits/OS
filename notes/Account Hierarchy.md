---
tags: [gtm-os, data-model, concept]
status: growing
created: 2026-10-28
updated: 2026-10-28
aliases: [parent account, subsidiaries, rollup, self-referencing relationship]
---

# Account Hierarchy

Companies own companies: Alphabet owns Google and YouTube. The model has to show that without storing anything twice.

## Where a deal belongs
**Q:** Someone at YouTube buys from you. Whose account does the deal belong to?
**A:** **YouTube's**, the account that actually bought. Not Alphabet's, and **not both**. Storing it on both would double-count revenue.

## How the tree is stored
- The account table gets a **parent account** column that points to *another row in the same table*. That's a **self-referencing relationship**, and together the accounts form a **tree**.
- Data is stored **once, at the right level** (the deal on YouTube); the hierarchy supplies the bigger picture.
- "How much revenue from Alphabet?" = add up everything in the tree beneath it. That's a **rollup**.

```
Alphabet
├── Google
└── YouTube ── deal #42 (stored here only)
```

## When the parent changes (acquisitions, divestments)
**Q:** Alphabet sells YouTube to another company. What happens?
**A (right):** YouTube's deals and history stay on YouTube; its parent becomes the new owner.

> [!warning] Correction: don't overwrite the parent column
> If you just replace "Alphabet" with the new owner, every *past* YouTube deal now rolls up to the new owner, and **Alphabet's historical revenue silently drops**. History rewrites itself.

**Fix:** the parent link is a **dated relationship**, with a start and an end date: Alphabet until the sale, the new company from then on. A report for any period uses **whoever owned it at the time**.

## The pattern you're now seeing
This is the third time the same rule has appeared: a person's employer, a deal's stage, and now an account's parent. **Relationships that change over time get dates, not overwrites** ([[Temporal Modelling]]).

Related: [[Core Data Model]], [[Person vs Account]].
