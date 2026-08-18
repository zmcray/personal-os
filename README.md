# Personal OS

A personal AI operating system I designed and built on Claude (Cowork, Anthropic's attended desktop-agent surface, plus Claude Code for terminal work). I run McRay Group, a small AI-native consulting company; this system is its back office. It runs my working day end to end: morning launch, call prep, capture routing, CRM hygiene, memory consolidation, evening synthesis, weekly planning, and a monthly audit of itself. The operating rule behind all of it: when a problem repeats, the fix is an agent or a closed automation loop, not more of my time. Headcount here is 0. The system runs the back office a small ops team would otherwise carry, and once a month it audits itself for drift.

This repo is the public, sanitized cut. The live system has 53 custom skills across 13 closed loops, integrated with 12 external systems (Supabase, Notion, Linear, Attio for CRM, Gmail, Google Calendar, Granola for call transcription, n8n for always-on workflow automation, Obsidian, and others). What you see here is the architecture documentation plus 12 flagship skill definitions. Private data... contact records, database IDs, emails, business strategy... is scrubbed or excluded entirely. Placeholders like `<SUPABASE_PROJECT_ID>` mark where live values sit in the real system.

## How it runs a day

At 6 AM, `begin-the-day` boots the system: drains overnight mobile captures, pulls the live calendar, builds call-prep briefs from the CRM, and reads the focus file the previous evening wrote. Mid-day, `pulse` scans 4 sources and stays silent unless something needs attention. At 4:45 PM, `end-of-day` sweeps every session transcript from the day into a ledger, closes the roadmap loop, and hands off to `advisor`, which tracks commitments in both directions, ranks tomorrow's uncertainties, and dispatches capped overnight research agents. At 2 AM, `memory-sync` runs a decay-and-dedupe pass on the system's living memory, pruned to a hard 200-line budget.

The self-feeding part is the design center: evening writes what morning reads, weekly review sets the lens daily briefings use, and a monthly audit (`mercer`, 8 modules) checks the whole thing for drift... including whether its own loops are still closed.

## Architecture

Everything is built on one rule: a thing is either *thinking* (markdown in a structured folder tree) or a *system of record* (an external tool that owns that domain). Attio owns relationships. Linear owns build state. Supabase owns execution. The calendar is always pulled live, never cached. The folder tree never duplicates a system of record.

The system also runs across 3 trust-separated surfaces: 2 attended (knowledge work, coding) with high-trust credentials, and 1 autonomous... a headless always-on agent (Argus) that processes untrusted input unattended and is therefore deliberately low-trust: no filesystem access, no table grants, RPC-only database writes with quarantine lifecycles, staged privilege granted on observed behavior, and a mutual heartbeat watchdog between the surfaces. `docs/surfaces-and-trust.md` covers the threat model; `docs/memory-substrate-pattern.md` has the full implementation, down to the SQL.

Every automation loop must pass through 6 stages... trigger, capture, process, store, review, close... and `docs/loops-map.md` audits all 13 loops against that template, with an explicit list of what's broken. The system documents its own failure modes.

Key docs:

- `docs/surfaces-and-trust.md` ... the 3-surface trust model for attended vs. autonomous agents
- `docs/memory-substrate-pattern.md` ... two-store memory with single-writer discipline, RPC-only agent access, async embedding backfill
- `docs/loops-map.md` ... the 13 closed loops, audited stage by stage
- `docs/loop-engineering-assessment.md` ... the OS benchmarked against external agentic-loop frameworks, with scores
- `docs/os-architecture.md` ... folder architecture, data flows by time of day, naming discipline
- `docs/knowledge-capture-pipeline.md` ... the signal ETL pipeline (X bookmarks, podcasts, web clips → classify → vault → wiki)
- `docs/supabase-roadmap-contract.md` ... the data contract for multi-agent writers against the execution database
- `docs/skill-conventions.md` ... packaging, eval, and lifecycle discipline for the skill layer
- `docs/sample-output-audit-history.md` ... one real, redacted artifact from the running system: the monthly audit's trend file, including the entry where the auditor itself failed and what fixed it

## The skills

Each folder under `skills/` holds a `SKILL.md`: the definition Claude loads to run that capability. The live versions also carry instruction files, templates, and eval checklists; those are excluded here because they contain worked examples with real data.

Highlights: `mercer` (the system audits itself monthly), `memory-sync` (self-improvement under a hard budget cap), `day-ledger` (cross-session transcript sweep with two-way database sync), `caspian` (a multi-phase product-strategy council... 5 personas, red-team and eng-review passes... that writes PRDs to disk, Linear, and Notion in one pass), and `skill-creator` (Anthropic's stock meta-skill for building and evaluating skills, extended here with a model-policy gate and a portable packaging step; included to show the eval discipline the rest were built under, not claimed as original work).

## What's not here

The private half: CRM data, people profiles, investment thesis work, client deliverables, and every skill output. This repo shows how the machine is built, not what it produced.

Built and maintained by Zack McRay. Not accepting contributions... this documents a live personal system. Fork the ideas freely.
