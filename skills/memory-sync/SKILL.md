---
name: memory-sync
description: >
  Daily learning loop that processes the day's activity and consolidates learnings into a living memory file. Runs two scheduled passes: (1) Daily pass at 6 PM ET (orient > gather > consolidate > prune to 200-line budget), (2) Overnight pass at 2 AM ET (decay > dedupe > pattern detect > rehearsal promotion) so memory learns while Zack sleeps. Can also be triggered manually with "/memory-sync", "sync memory", "consolidate memory", or "overnight pass".
---

# Memory Sync

## Model Policy (2026-07-05)
Both passes (6 PM daily, overnight) run at Sonnet capability. No subagents needed. Do not escalate to Opus/Fable for scheduled runs.

You are running Zack McRay's memory consolidation loop. This is how the system learns. Two passes per day. The 6 PM daily pass processes new activity into memory. The 2 AM overnight pass tightens what's already there (decay, dedupe, pattern detection, rehearsal promotion) so memory is sharper every morning than it was the night before.

**Tone:** Internal system process. No chat output unless Zack triggered it manually. If manual, deliver a lean summary of what changed.

**Which pass?** Detect the trigger:
- Daily pass (6 PM ET, or manual "sync memory" / "/memory-sync" during the day): run Phases 1-4 below.
- Overnight pass (2 AM ET, or manual "overnight pass" / "consolidate overnight"): run Phase 5 only. Do NOT run 1-4 at 2 AM... the daily pass already did that.

---

## Workflow: Daily Pass (6 PM ET)

### Phase 1: Orient
Read `instructions/orient.md`. Load current memory state, flag anything stale or drifting.

### Phase 2: Gather
Read `instructions/gather.md`. Pull signal from all available sources: session transcripts, feedback log, Notion activity, file changes, calendar, and (added 2026-07-22) Supabase `os.memory_observations` (last 36 hours) plus `os.captures` activity counts for the briefing line.

### Phase 3: Consolidate
Read `instructions/consolidate.md`. Rewrite `40_OS/08_Memory/active-memory.md` with everything learned. Also (added 2026-07-22) write back 0-5 durable, self-contained observations per day to `os.memory_observations` with `source = 'cowork'` (see consolidate.md step 6). Observations are an input and an output of this pass, never a bypass around active-memory.md's single-writer rule. Use absolute dates. Every new entry carries `[w:N]` and `[src: <tag>]`. Follow the template in `templates/active-memory-template.md`.

### Phase 4: Prune
Read `instructions/prune.md`. If active-memory.md exceeds 200 lines or ~25kb, archive overflow to `40_OS/08_Memory/memory-archive-YYYY-MM.md`, then compress.

---

## Workflow: Overnight Pass (2 AM ET)

### Phase 5: Overnight Consolidation
Read `instructions/overnight.md`. Six sub-phases... pipeline health check > decay > dedupe > pattern detect > rehearsal promotion > log rotation (monthly GC). Pipeline health check (5.0) runs first and cross-references Granola (MCP) + Calendar + Attio debrief notes to catch capture failures and stuck post-call-debrief handoffs (Attio is the sole debrief record as of 2026-07-05; the Notion Meeting Notes database is retired), plus Check C (added Step 9, 2026-07-22): the Argus mutual watchdog... upserts the Mac's beat into Supabase `os.heartbeats` and flags (overnight-log + one Slack alert line) if Argus's hourly beat is >24h stale. Phases 5.1-5.4 run ONLY against `active-memory.md` and the last 7 days of transcripts. Does NOT re-gather new signal. Writes a "Patterns Observed Overnight" block + pipeline health status to `40_OS/08_Memory/overnight-log.md` that `begin-the-day` reads and surfaces in the morning brief. Log rotation (5.5) runs last and only on the FIRST overnight pass of each month: rotates previous-month entries out of `overnight-log.md` and `email-triage-log.md` into sibling `*-archive-YYYY-MM.md` files (mirrors the memory-archive-YYYY-MM.md pattern), keeping each active file's header + current-month entries only.

---

## Output Format

**If scheduled (no user present):** Silent. Write files, done.

**If manual trigger (daily pass):** One summary block in chat:

```
Memory Sync Complete (2026-04-02)
- Gathered: X sessions, Y feedback entries, Z Notion changes
- Added: [brief list of new learnings]
- Pruned: [N lines archived / nothing pruned]
- Flags: [anything stale or needing attention, or "none"]
```

**If manual trigger (overnight pass):** One summary block in chat:

```
Overnight Pass Complete (2026-04-02)
- Decayed: [N entries weight-reduced]
- Merged: [N duplicate pairs collapsed]
- Patterns: [brief list of patterns surfaced for morning brief]
- Rehearsed: [N entries promoted by touch-count]
```

---

## Key Rules
- Every entry in active-memory.md gets an absolute date (YYYY-MM-DD). No "recently", "last week", "a few days ago".
- **Every new entry gets a weight tag `[w:1-5]` and a provenance tag `[src: <kebab-case>]`.** Schema is documented at the top of active-memory.md. Weight defaults to `w:3` unless context implies higher. Provenance defaults to `[src: manual]` only when no traceable source exists. Format:
  ```
  YYYY-MM-DD [w:N] [src: <tag>]: entry text.
  ```
- **Weight scale.** `w:5` = preserve forever (thesis anchors, identity). `w:4` = critical path (P1, blockers). `w:3` = default working context. `w:2` = background. `w:1` = ephemeral.
- **Backfill is opportunistic.** Do NOT rewrite legacy entries (pre-Phase-B) in a big bang. Only add tags when an entry is touched, updated, or promoted during consolidation.
- Never delete or modify CLAUDE.md structure. CLAUDE.md is the stable index. active-memory.md is the living document.
- Archive before pruning. Nothing is truly lost.
- If a learning contradicts something in CLAUDE.md or 00_Context files, flag it... don't silently override.
- Prioritize retention of: corrections/preferences Zack expressed, people context, project status changes, patterns observed across sessions.
- Deprioritize: routine task completions, one-off questions, anything already well-documented in existing memory files.
- No em dashes. Use commas, periods, semicolons, or "..." instead.

---

## Reference Docs
- `00_Context/os-config.md` ... canonical config (database IDs, emails, paths, rules)
- `shared/folder-paths.md` ... all local folder paths
- `shared/formatting-rules.md` ... formatting standards
- `shared/notion-databases.md` ... database IDs
