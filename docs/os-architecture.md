# OS Architecture

Last regenerated: 2026-07-06, supersedes the 2026-04-08 Workflows snapshot.

Purpose: this is the map of how Zack's Work folder is shaped and how work moves through it. It describes structure, not configuration. It answers "where does this belong" and "what talks to what," not "what's the database ID." For that, go to `00_Context/os-config.md`. For the index of everything else, go to `CLAUDE.md`.

---

## The Two-Things Principle

Everything in Zack's world is one of two things: thinking, or a system of record. The Work folder holds the first. It never tries to be the second.

**Work folder = thinking + OS machinery.** Thesis development, strategy notes, brand voice, skill logic, memory, vault, call prep docs, project registries as local mirrors. This is where Claude reads and writes when the work is reasoning, drafting, or remembering.

**Systems of record live elsewhere, and the Work folder points to them instead of duplicating them:**
- Code → `~/Developer/<repo-name>/` (flat, kebab-case, GitHub-backed). Never in Work, never in an iCloud path. iCloud evicts `.git` and `node_modules` and breaks builds.
- Relationships → Attio (CRM). Contact history, follow-ups, debriefs all live as Attio notes and tasks.
- Build state → Linear (Mcraygroup team). Pipeline stage, features, issues.
- Execution → Roadmap (Notion at the time of this snapshot; migrated to Supabase 2026-07-21, see `supabase-roadmap-contract.md`). Daily/weekly task state.
- Calendar → Google Calendar. Live pull, never cached as the source of truth.

When a new need shows up, the first question is which of these two buckets it belongs to. If it's a system of record, it does not get a Work-folder home of its own; it gets a pointer (a local mirror file at most, refreshed on a schedule, never hand-edited as canonical).

---

## Top-Level Map

**`00_Context/`** ... Identity, voice, and behavioral rules. Who Zack is, how Claude should act, and the canonical config values (database IDs, emails, paths, calendar rules). Every session reads this first. The canonical file list (the allowlist) lives in `00_Context/README.md`. Does NOT hold: handoffs, briefs, session logs, checklists, skill logic, memory entries, or project content, those live in `40_OS/` (handoffs → `16_Handoffs/`, process/architecture docs → `01_Workflows/`, logs/memory → `08_Memory/`).

**`01-mcray-group/`** ... The firm itself. Thesis, sourcing, deal work, models, ops, value creation, partners, legal, accounting, capital, strategy, events, branding, and the company vault (`50-vault/`). This is McRay Group's operating home, not a project folder. Grandfathered kebab-case naming (see Naming Conventions below). Does NOT hold: personal career materials or unrelated side projects.

**`20_Career/`** ... Zack's personal career materials: resumes, cover letters, brand content, interview prep. Independent of McRay Group. Does NOT hold: McRay Group deliverables or client work.

**`30_Projects/`** ... Non-code projects only: learning tracks, research efforts, anything that isn't a software build. Code-based builds do NOT live here anymore, they live in `~/Developer/`. This folder is a holdover from the pre-migration model for projects that were never code (e.g. AI_Learning) plus a couple of client engagements not yet re-homed.

**`40_OS/`** ... The operating system itself: skills, memory, automations, templates, vault, dashboards, call prep, migrations. This is the machinery that runs Zack's day. See interior map below.

**`99_Archive/`** ... Dead weight with a paper trail. Anything retired, superseded, or wound down moves here into a dated subfolder rather than being deleted. Nothing gets deleted without explicit approval; this is the holding pen instead.

---

## 40_OS Interior Map

- **`01_Workflows/`** ... process documentation, component docs for how pieces of the OS work together. Holds `loops-map.md` (the index/map of all closed loops) and `loops/` (per-loop workflow detail).
- **`02_Agents/`** ... agent definitions and persona configs (Hagen, Mercer, Luce infrastructure where it lives outside the skill folder itself).
- **`03_Automations/`** ... automation specs and configs that aren't full skills (n8n references, scheduled-task adjacent material).
- **`04_Prompts/`** ... standalone prompt library, reusable prompt fragments not packaged as skills.
- **`05_Skills/`** ... the skill library. Every skill's source lives here; `dot_skills/` is the packaged-for-install cache.
- **`06_Templates/`** ... reusable document and file templates (PROJECT.md template, note templates).
- **`07_Feedback/`** ... the feedback log (`log.md`). Corrections, gotchas, learnings captured via `/flag-that`.
- **`08_Memory/`** ... deep reference layer: active memory, archives, people profiles, glossary, workstreams, scheduled tasks, commitments, self-model.
- **`09_Vault/`** ... the personal thinking vault (compile-forward: raw > wiki > outputs).
- **`10_Migrations/`** ... migration scripts and runbooks for structural moves (Attio migration, Developer folder migration, etc.).
- **`11_Best Practices/`** ... the Best Practices database mirror, patterns worth repeating.
- **`12_Call_Prep/`** ... call prep docs (.docx), with `_archive/YYYY/` for anything older than 30 days.
- **`13_Dashboards/`** ... visual dashboards and HTML artifacts (this doc's companion visual lives here).
- **`14_Plugins/`** ... plugin-specific material (e.g. client-specific session-script packs).
- **`15_Tooling/`** ... tooling references and utility scripts.
- **`16_Handoffs/`** ... active, in-flight handoffs and bridge docs (the notes one session writes to hand work or context to the next). Active only; completed handoffs move to `99_Archive/`. Handoffs go here instead of `00_Context/`; see `16_Handoffs/README.md` for what belongs.

---

## Data Flows

**Morning.** `begin-the-day` runs first (6 AM ET auto-fire or on-demand). Layer 0.25 auto-runs `sync-captures`, draining any Pending rows in the Notion Capture Inbox (mobile/web overnight captures) into the right vault file before the briefing reads from those files. Calendar pull, Roadmap check, call prep hand-off to `daily-call-prep`, heartbeat check on scheduled automation.

**Capture.** Quick-capture skills write to different homes depending on type:
- `/thesis`, `/strategy`, `/brand` → McRay Group vault `thinking/` files (desktop) or Notion Capture Inbox tagged Pending (mobile/web, drained by `sync-captures`).
- `/roadmap` (and `/task`) → Notion Roadmap. Tasks and ideas, not builds.
- `/buildnote` → Linear (Mcraygroup team), Backlog state, tagged to the resolved Linear Project. OS-flavored buildnotes route to Notion Cowork OS Backlog instead. Buildnotes bypass the Roadmap hub-and-spoke entirely.

**Evening.** `end-of-day` runs the full wrap sequence: invokes `day-ledger` (sweeps the day's session transcripts into a synthesized ledger grouped by project, writes to `40_OS/08_Memory/daily-ledger/`), checks for undebriefed calls, closes the Roadmap loop on today's tasks, asks for tomorrow's deep-work flag, then invokes `advisor` for the evening chief-of-staff pass (commitments tracking, decision queue, plan-vs-reality delta, horizon scan, tomorrow's call, capped overnight research). `memory-sync` daily pass (6 PM ET) consolidates the day into `active-memory.md`, pruned to a 200-line budget.

**Weekly.** `weekly-review` splits into a headless RETRO (Friday ~11:45 AM, compiles the week, drafts next week's plan) and an attended CONFIRM (12:30-1:00 PM, ratifies priorities and approved proposals). Feeds `next-week-plan.md`.

**Monthly.** `mercer` runs the full workspace audit (1st of the month): file hygiene, memory drift, skill health, closed-loop analysis, system health, trend tracking, wiki health, model selection review. Writes to `40_OS/08_Memory/audit-history.md`.

**Overnight.** `memory-sync` overnight pass (2 AM ET) runs decay, dedupe, pattern detection, and rehearsal promotion on top of the daily pass.

---

## Naming Conventions

Top-level folders use `NN_Title` numbering (`00_Context`, `40_OS`, etc.). `01-mcray-group/` is the one documented exception: fully kebab-case throughout, grandfathered because too many live pointers reference it to safely rename. Do not rename it and do not create a second kebab-case folder elsewhere without the same level of justification.

**Archive discipline:** nothing in `99_Archive/` gets deleted. Every retirement gets a dated subfolder (`99_Archive/<what>-<reason>-YYYY-MM-DD/` or `-retired-YYYY-MM-DD`) so the reason and the date travel with the files. This applies to skills, workflows, folder structures, anything wound down. Files never disappear without explicit approval; archiving is the default disposition for "this is done."

---

## Pointer Etiquette

`CLAUDE.md` is the index. It does not contain detailed instructions, it points to where everything lives, and every session reads it first.

`00_Context/os-config.md` is canonical config: database IDs, emails, folder paths, calendar rules, tactical rules, model selection tiers. If a skill needs an environment-specific value, it reads from there, not from this document.

This document describes shape: what exists, what belongs where, how work flows between pieces. It intentionally does not duplicate database IDs, email addresses, or other config values already living in `os-config.md`. If those two ever appear to disagree, `os-config.md` wins on values and this document wins on structure.
