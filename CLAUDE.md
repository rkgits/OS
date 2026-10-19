# Repo instructions

This repo is Reza's research vault for designing a company "OS" (GTM data integration, ingestion, identity resolution, workers/agents, and eventually actions from a dashboard). It is meant to be opened as an **Obsidian vault**.

## Note format (required for every document in this repo)

- **Small notes, one concept each**, in `notes/`. File name = note title (spaces allowed, e.g. `Identity Resolution.md`).
- **YAML properties block** at the top of every note:
  ```yaml
  ---
  tags: [gtm-os, <area>]
  status: seed | growing | evergreen
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  aliases: [...]   # optional
  ---
  ```
- **Link with wikilinks**: `[[Note Title]]`, `[[Note Title#Heading]]`, `[[Note Title|label]]`. Every note links to at least one related note.
- The hub is `notes/GTM OS - Map of Content.md`. Add every new note to it, and keep its "Current position" section accurate.
- Use Obsidian callouts where useful (`> [!warning]`, `> [!todo]`), and `- [ ]` tasks in `Open Threads.md`.

## Knowledge here, work there

Two repos, two jobs:

- **`rkgits/OS` (this repo)** is the **knowledge base**: concepts, decisions and the reasons behind them, corrections, examples. Notes only.
- **`galliot-us/GTM-Engine`** is the **system of record for work**: code, plus **GitHub Issues** for tickets and tasks.

Tickets and task lists never live in this repo. When a discussion produces build work, create an issue in GTM-Engine and link to it from the relevant note (e.g. `[GTM-Engine #1](https://github.com/galliot-us/GTM-Engine/issues/1)`). `Open Threads.md` holds learning and design questions only.

## Keeping it current

- This repo is the single source of truth. An older Claude Docs copy (claude.ai artifact d97bce51-deb7-437d-853b-4f115289ddaa) is frozen; do not update it.
- Whenever a session continues the discussion, capture the new material before finishing: extend the relevant notes or add new ones, update `Open Threads.md` (move resolved items to "Resolved"), and bump `updated`.
- Keep the voice: precise terminology in **bold**, corrections stated plainly, decisions marked explicitly.
- Commit with a short message describing what was added, and push to `main`.
