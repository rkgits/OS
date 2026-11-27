---
tags: [gtm-os, interface, design, decision]
status: seed
created: 2026-11-26
updated: 2026-11-26
aliases: [design system, design tokens, dashboard design, UI library]
---

# Dashboard Design System

**One set of rules, tokens and components that every future page, report and dashboard reuses**, so the interface ([[Dashboard Architecture]], layer 5 of the [[System Map]]) stays consistent as it grows. Same idea as [[Core Plus Extensions]] for data: a stable core everything else plugs into.

## Name it properly
What was described ("a library of assets, rules and elements for every future page") is a **design system**. It's built in four layers, in this order, each referencing the one before:

1. **Principles**: a one-page statement of how the product should feel and behave.
2. **Design tokens**: the raw decisions as named values: colours, type sizes, spacing, corner radius, shadows. Components never hard-code a colour; they use a token.
3. **Components**: badges and tags, tooltips, tables, cards, buttons, inputs.
4. **Patterns**: how a page, a report or an edit-and-apply flow is put together from components.

**Decision: don't start from scratch.** Build on **shadcn/ui** with **Tailwind**: you own the component code (it's copied into your repo, not a black-box dependency), and AI coding tools know it well.

## Principles (agreed)
1. **Light and calm.** Light background, soft colours; strong colour is kept for meaning, not decoration.
2. **Details on demand.** The surface stays clean; hovering or clicking reveals more. **Nothing important exists only on hover.**
3. **Colour means something.** Each tag colour stands for one status or category, used the same way everywhere.
4. **Every action shows its state.** Pending, saving, saved or failed: always visible, never silent.
5. **Changes are safe.** Edits can be reviewed before they're applied, and undone where possible.
6. **Dense but scannable.** Lots of information in a clear order of importance, so the eye knows where to land.

## Corrections
> [!warning] Hover-only details fail
> Touch screens have no hover, and keyboard users never trigger it. Anything shown on hover must also open on **click or focus**. (The font test page does this: tags open their tooltip on hover, tab and click.)

> [!warning] "Wait, then apply" is a design decision of its own
> An interactive dashboard where an edit takes a moment to apply needs its states designed up front: **pending → saving → saved**, or **failed** with a way to retry. Leaving them to each page is how you get spinners that never stop and failures nobody sees. It's the interface version of **fail loudly, not silently** ([[Recurring Lessons]]). On the engine side, every such action must also be made idempotent ([[Idempotency]]), because a user *will* click Apply twice.

> [!warning] Glass is for a few surfaces, not for data
> The "liquid glass" style (Apple's translucent look, used on the LSTC site) reads well on marketing pages but makes dense data harder to read. Use it for a **few surfaces**: the navigation bar, a dock, pop-up panels. Tables and charts sit on **solid, light** backgrounds.

## Tokens: decisions so far
### Typography
**Decision: Inter** for all interface text (chosen Nov 26 after a side-by-side test against Geist and IBM Plex Sans). Inter is the proven default for product UI and stays very legible at 12–13px, where tags, table headers and tooltips live.
- **Tabular figures in every numeric column** (`font-variant-numeric: tabular-nums`): all digits the same width, so columns of numbers line up.
- **Weights:** 400 for body, 500 for labels, 600 for titles. Three weights, used consistently.
- **Monospace for IDs, event names and code:** Geist Mono, as tested beside Inter. *(Tentative; swap if it doesn't sit well.)*

### Colour
**Decision (Nov 26): the main accent is the Galliot logo blue, `#179CFF`** (taken from the logo SVG on galliot.us; the wordmark is grey `#5A5A5A`). Green and yellow are **chosen for readability, not to match the brand**: clearly different in hue from the blue, so a status never looks like a link.

> [!warning] Correction: the brand colour can't be the text colour
> `#179CFF` on white is **2.9:1**, below the 4.5:1 that readable text needs. So the logo blue is used where it's a *fill*: highlights, icons, chart accents, the selected item's mark. Text and buttons use **darker steps of the same hue**. Same rule for yellow: yellow text on white is unreadable (1.8:1), so a yellow tag is **dark amber text on a pale yellow background**.

| Token | Hex | Use | Contrast on white |
| --- | --- | --- | --- |
| Brand blue | `#179CFF` | highlights, icons, chart accent, focus ring | 2.9 (fill only) |
| Button blue | `#0A6FD6` | primary buttons (white text), links | 4.9 |
| Blue text | `#085BB5` | text on pale blue | 6.6 |
| Green fill | `#16A34A` | dots, bars, success icons | 3.3 (fill only) |
| Green text | `#15803D` | success tag text | 5.0 |
| Yellow fill | `#F5B800` | dots, bars, highlights | 1.8 (fill only) |
| Yellow-tag text | `#7A4F00` | pending tag text on `#FEF4D6` | 6.5 on its tint |
| Red | `#B42318` | failed | 6.6 |
| Logo grey | `#5A5A5A` | secondary text | 6.9 |

The pattern behind it: **every hue gets a *fill* step and a *text* step.** A brand colour is chosen for recognition; a text colour has to pass a contrast check. One colour rarely does both.

### Icons
**Decision: use an existing open-source set, don't draw our own.** An icon set is hundreds of drawings that must share one grid, stroke and style; making that is a design project of its own. Candidates compared side by side in a test page with 37 dashboard meanings: **Lucide** (shadcn/ui's default, ISC), **Phosphor** (six weights, MIT), **Tabler** (largest, MIT), **Heroicons** (polished, smallest, MIT).

**Decision (Nov 26): Lucide.** It ships with shadcn/ui, so every component we add already uses it; any other set would mean swapping icons inside each component. Its 2px rounded line matches Inter's weight at 16–20px, and at about 2,100 icons it covers a dense dashboard.

> [!todo] Not decided yet
> Spacing scale and corner radius; shadows; dark mode or not; and **where the design system lives** (its own repo, or a folder in the dashboard's repo). See [[Open Threads]].

## Where we are
Principles, the approach, the font, the colours and the icon set are set; density (spacing, corners, shadows) is next. The order from here: finish tokens (exact colours, then spacing and radius), then the core components (tag, tooltip, table, card, button, and the apply-state pattern), then page patterns.
