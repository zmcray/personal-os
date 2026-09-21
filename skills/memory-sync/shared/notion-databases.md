# Databases

Refreshed 2026-07-27. Filename kept as `notion-databases.md` because 8 skills point at it, but the scope is now every system of record, not just Notion. Notion is no longer the default home for anything operational.

**This file is deliberately thin.** Which system owns what, and what is frozen. Full detail and IDs live in the canonical files.

| For | Read |
|---|---|
| Full database + project table | `00_Context/databases.md` |
| Config values and data source IDs | `00_Context/os-config.md` §2b |
| Supabase Roadmap schema contract | `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md` |

---

## Who Owns What

| Domain | System | Notes |
|---|---|---|
| Relationships, contacts, debriefs, follow-up tasks | **Attio** (workspace McRayGroup) | Objects `people` + `companies`. Sole CRM and sole system of record for call debriefs. |
| Task/production execution | **Supabase** `os` schema, project `mtypgfwcebzsdlbuojef` | `os.roadmap_items`, `os.workstreams`, `os.roadmap_events`, `os.work_threads`, `os.captures`, `os.memories` (write via `os.memory_write`), `os.heartbeats`. |
| Build state, features, product pipeline | **Linear** (Mcraygroup team) | One Project per build, Issues = features. `/buildnote` writes here. |
| Portfolio index, capture inbox, content calendar, signal | **Notion** | See active list below. |
| Thinking, decisions, firm knowledge | **Markdown** (vault folders) | `40_OS/09_Vault/` personal, `01-mcray-group/50-vault/` firm. |

## Active Notion Databases

| Database | Purpose | Used by |
|---|---|---|
| **Content Calendar** | Posts (planned/ready/published) + engagement metrics | content-engine, content-reply, content-performance |
| **Signal Library** | Captured and classified external signal | signal-classify-sync, signal-session, begin-the-day |
| **Capture Inbox** | Mobile/web captures pending routing | sync-captures |
| **Project Registry** | Portfolio-level index of all projects | Cowork orientation, caspian |
| **Personal OS Build Log** | Every component in the OS | doc-that |
| **Best Practices** | Logged patterns and lessons | best-practice |
| Product backlogs (Forge, Pulse, Cowork OS) | Per-product capture layers | buildnote (Cowork OS variant) |
| McRay Group Roadmap | Firm-level planning layer | weekly-review |

Data source IDs are in `os-config.md` §2b. Do not hardcode an ID from memory; read it.

## FROZEN / Retired ... Never Query These

| Database | Status |
|---|---|
| Notion **Roadmap** + **Workstreams** | **FROZEN 2026-07-22.** Migrated to Supabase `os` schema. ARCHIVED prefix, read-only, retained for provenance only. All roadmap reads and writes go to Supabase. |
| Notion **CRM** | **Archived 2026-06-10.** Replaced by Attio. Never query. |
| Notion **Meeting Notes** | **Deprecated 2026-07-05.** Attio notes are the sole debrief record. |
| Notion **Build Notes** | Archive-only since 2026-04-28. `/buildnote` writes to Linear. |
| **Thesis Scratchpad** page | Deprecated, migrated into the vault. |
| **Learn Plan** | Status unclear, likely dead. Its only documented consumer was `daily-pe-learning`, a skill that no longer exists. The live path is `learn-capture` (`/learn`), which appends to `40_OS/09_Vault/raw/learn-queue.md` and touches no Notion database. Do not write to Learn Plan without checking with Zack. |

## Known Stale Pointers Elsewhere

Flagged 2026-07-27, not yet fixed at the source. Do not trust these if you encounter them:

- `os-config.md` §2b still lists `daily-pe-learning` as the consumer of Learn Plan. That skill does not exist.
- `databases.md`'s "Local projects" table lists projects under `30_Projects/Clients/` and `30_Projects/McRayGroup/` paths that no longer exist. Actual contents of `30_Projects/`: AI_Learning, Grove Grocer, acquired-taste, cos-agent, dfw-paving. Use the Project Registry or `00_Context/active-projects.md` instead.
- `databases.md` says the Thesis Scratchpad migrated to vault `Inbox/`, a folder archived on 2026-04-09. Content is in `raw/`.
