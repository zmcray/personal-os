# Phase 5: Overnight Consolidation

**Purpose:** While Zack sleeps, tighten what's already in active-memory.md so he wakes up to sharper memory. The daily pass writes new signal. This pass refines it.

**Runs at 2 AM ET**, scheduled. Does NOT re-gather new signal. Works entirely on what the 6 PM daily pass already wrote.

**Scope:** `40_OS/08_Memory/active-memory.md` plus the last 7 days of session transcripts and Granola meeting notes.

**Output:** Rewritten `active-memory.md` (same file, tightened in place) plus an append to `40_OS/08_Memory/overnight-log.md` capturing what changed and any patterns detected. `begin-the-day` reads the overnight-log the next morning.

---

## Six Sub-Phases

Run in order. Sub-phase 5.0 runs first as a pipeline health check. Sub-phases 5.1-5.4 modify active-memory.md in place; always re-read the file before starting the next sub-phase. Sub-phase 5.5 runs last and only fires on the first overnight pass of a new month.

### Sub-Phase 5.0: Debrief Pipeline Health Check (Attio-native, reworked 2026-07-05)

Added 2026-04-09 after the Granola > Notion n8n workflow was discovered silently inactive for 3 days while the overnight pass inferred backlog risk from memory gaps instead of checking source of truth. Reworked 2026-07-05: Attio is the sole system of record for debriefs and the Notion Meeting Notes database is retired... never query it.

This sub-phase runs BEFORE the memory work and writes directly to overnight-log.md (never to active-memory.md). Its job is to catch pipeline failures at their source so the morning brief can surface them loudly.

**Two checks.**

**Check A: Capture freshness.** Query the Granola MCP (`list_meetings` / `get_meetings`) for meetings in the last 24 hours. Count them. Then query Google Calendar (`zack@mcraygroup.co`) for external calls/meetings in the same window (non-blocking events, external attendees). If the calendar shows N external meetings and Granola captured zero, Granola capture is broken. Flag in overnight-log.md under a new `### Pipeline Health` section.

**Check B: Debrief staleness.** For each external call in the last 72 hours (from calendar + Granola), check Attio for a debrief note on the counterparty (search-notes-by-metadata on the person record) dated on/after the call. Calls older than 48 hours with no Attio debrief note represent EOD > post-call-debrief handoffs that silently failed. List them in overnight-log.md under `### Pipeline Health` with meeting title, date, and counterparty so the morning brief can surface them individually.

**Check C: Argus mutual watchdog (added Step 9, 2026-07-22).** Argus (the Hetzner CoS agent) beats an hourly heartbeat into Supabase `os.heartbeats` (row `source='argus'`, project `mtypgfwcebzsdlbuojef`). This check is the durable Mac side of the mutual watchdog... it lives here (not only in begin-the-day) because this pass runs daily on the Mac forever, while the Cowork morning-launch twin is disabled after the Argus morning-brief cutover. One `execute_sql` call:

```sql
with up as (
  insert into os.heartbeats (source, beat_at, note)
  values ('mac', now(), 'memory-sync overnight')
  on conflict (source) do update set beat_at = now(), note = excluded.note
)
select beat_at,
       round((extract(epoch from (now() - beat_at))/3600.0)::numeric, 1) as age_hours
from os.heartbeats where source = 'argus';
```

The `up` CTE writes the Mac's own beat (which Argus's daily 7 AM heartbeat job reads in the other direction). If `age_hours` > 24 or the row is missing, Argus's hourly cron is dead or the box is down.

**Severity rules.**
- Zero Granola captures + 1+ external calls in last 24h = CRITICAL. Surface as top item in morning brief.
- 1+ external calls older than 48h with no Attio debrief note = HIGH. Surface as list in morning brief.
- Argus heartbeat >24h stale or missing = HIGH. Write to `### Pipeline Health` with the last beat time in ET, AND post one alert line to `#argus` (C0BJUQBEWPK) via the Slack tool if available: `WATCHDOG: Argus heartbeat stale since [time ET] ... check box (ssh argus, docker ps, hermes cron status).` The Slack post matters because after cutover the morning brief itself comes from Argus... if Argus is down, overnight-log.md alone reaches no one.
- All clear = silent. Still write a one-line "Pipeline Health: OK (Argus beating)" entry to overnight-log.md for audit trail.

**Hard rule.** Sub-phase 5.0 is READ-ONLY against Granola, Attio, and Google Calendar. Do NOT create Attio notes or tasks, do NOT trigger post-call-debrief. Just report. The morning brief and EOD skill own the fixes. Two deliberate exceptions, both Check C: the `os.heartbeats` Mac-beat upsert (that IS the watchdog signal) and the single Slack alert line when Argus is stale.

**Why this check lives in overnight-pass and not EOD:** EOD runs at 4:45 PM when some of today's calls may not have finished their Granola processing yet. Overnight pass runs at 2 AM when everything should be landed and any gaps are real. Also, overnight pass is already the "what changed while I slept" layer, so pipeline breakage naturally belongs here.

### Sub-Phase 5.1: Decay

Adaptive weight decay. Low-weight entries fade faster than high-weight entries. This is the primary compression mechanism for the overnight pass.

**Rules:**
- `w:5` ... never decayed. Preserve forever.
- `w:4` ... decay only if older than 60 days AND the entry describes something that's been resolved (e.g., a completed P1, a shipped project).
- `w:3` ... default decay. Entries older than 30 days with no rehearsal (see 5.4) drop to `w:2`.
- `w:2` ... entries older than 14 days with no rehearsal drop to `w:1`.
- `w:1` ... entries older than 7 days with no rehearsal are candidates for removal in the next daily pass prune (but don't delete here... just mark).

**Do not:** change the text of any entry. Only adjust its `[w:N]` tag. If an entry has no weight tag (legacy), leave it alone.

**Record:** how many entries had their weight reduced. Log the section and short description in overnight-log.md.

### Sub-Phase 5.2: Dedupe

Find near-duplicates and merge them. Near-duplicate means: same topic, same entity, overlapping claims, but written on different dates or in slightly different words.

**Detection heuristic:** entries in the same section (e.g., both in People Updates) that share a named entity (person, company, project) and contain >60% semantic overlap.

**Merge rule:** keep the most recent entry. Bring any unique detail from the older entry into the newer one. Inherit the MAXIMUM weight of the two. Concatenate provenance tags if both have `[src:...]`: `[src: tag1, tag2]`.

**Conservative default:** if unsure, do NOT merge. Flag to overnight-log.md for Zack to review.

**Record:** how many pairs were merged, how many were flagged.

### Sub-Phase 5.3: Pattern Detection

Scan the last 7 days of:
- `active-memory.md` (all sections)
- Session transcripts from `/sessions/*/mnt/Work/` if available
- Attio debrief notes + tasks (via Attio tools: search-notes-by-metadata / get-note-body / list-tasks) and Granola meetings (via MCP: query_granola_meetings / get_meetings); local transcript archive at `40_OS/08_Memory/call-transcripts/` if deeper context is needed

Look for:
- **Recurring phrases** ... same idiom or concept appearing 3+ times across different sources.
- **Recurring entities** ... same person/company/project mentioned in 3+ different contexts.
- **Contradictions** ... two claims that can't both be true.
- **Rising themes** ... topics that grew in mention frequency over the past 7 days.

**Output format:** append to `40_OS/08_Memory/overnight-log.md`:

```
## Overnight Pass YYYY-MM-DD
### Patterns Observed

- **[phrase/entity/theme]** ... N occurrences across [source list]. [One-sentence interpretation.]
- ...

### Contradictions

- **[claim A]** (source X) vs **[claim B]** (source Y). [One-sentence note on what to resolve.]
- ...

### Rising Themes

- **[theme]** ... went from N mentions last week to M mentions this week.
- ...
```

If there are no patterns, write "No new patterns surfaced." Do NOT fabricate patterns.

**Record:** how many patterns surfaced.

### Sub-Phase 5.4: Rehearsal Promotion

The rehearsal effect: accessing a memory strengthens it. Entries that appear in multiple recent sessions get weight bumps; entries that only ever appeared once fade (handled in Sub-Phase 5.1).

**How to detect rehearsal:**
- An entry is "rehearsed" if its named entity (person, project, concept) appears in 2+ distinct sources across the last 7 days.
- Sources counted: session transcripts, Granola meeting notes, active-memory.md changes (daily pass diffs), feedback log, content calendar entries.

**Promotion rule:**
- `w:1` > `w:2` if rehearsed 2+ times.
- `w:2` > `w:3` if rehearsed 3+ times.
- `w:3` > `w:4` if rehearsed 5+ times AND the entry describes a critical path item, active commitment, or thesis anchor.
- `w:4` > `w:5` requires Zack's explicit approval. Do not auto-promote to w:5.

**Record:** how many entries were promoted. Append to overnight-log.md.

### Sub-Phase 5.5: Log Rotation (Monthly GC)

Runs LAST, and only on the FIRST overnight pass of each calendar month. Detection: if the most recent prior `## Overnight Pass` block in overnight-log.md is dated in an earlier month than today, this is the first pass of the month... rotate. Otherwise skip silently (no log line needed).

Two files, same procedure, mirroring the `memory-archive-YYYY-MM.md` pattern (precedent: `overnight-log-archive-2026-05.md`, `overnight-log-archive-2026-06.md`):

1. `40_OS/08_Memory/overnight-log.md` ... move all entries dated in the previous month (or older, if any strays remain) into `40_OS/08_Memory/overnight-log-archive-YYYY-MM.md`, where YYYY-MM is the month being rotated out.
2. `40_OS/08_Memory/email-triage-log.md` ... same, into `40_OS/08_Memory/email-triage-log-archive-YYYY-MM.md`.

**Rules:**
- Entry boundaries are the dated `##` headings (`## Overnight Pass YYYY-MM-DD` / `## Session: YYYY-MM-DD | ...`). Move whole blocks. Preserve formatting exactly, original physical order (even if entries were logged out of chronological order).
- Keep each active file's header (everything above the first entry) plus current-month entries only. Add a one-line blockquote pointer in the header noting what was archived, to which file, on what date.
- Archive file gets a short header: what it is, rotation date, what the live log retains, pointer to prior-month archives. If the target archive file already exists, append below its last entry instead of overwriting.
- This is a move, never a delete. Every entry survives somewhere.
- Note the rotation in THIS pass's overnight-log entry (one line under Notes: what rotated, to where, before/after line counts). This is the only exception to "do not add new entries"... the rotation note lives in the log, not in active-memory.md.

**Record:** before/after line counts for each rotated file.

---

## Hard Rules

- Do NOT add new entries in this pass. Only modify existing ones.
- Do NOT edit CLAUDE.md, 00_Context files, or people profiles.
- Do NOT re-gather signal. That's the daily pass's job.
- Do NOT delete entries. Worst case is weight reduction. Deletion only happens in the daily prune phase after archive.
- Preserve the schema header at the top of active-memory.md exactly.
- If the file is unchanged after all six sub-phases, still write a short "No changes this pass" entry to overnight-log.md with the date. Sub-phase 5.0 (pipeline health) always writes something, even if only "Pipeline Health: OK", because that line is an audit trail.

---

## Output to overnight-log.md

Always append, never overwrite. Each pass writes a dated block. `begin-the-day` reads the most recent block the next morning and surfaces the patterns section in the morning brief.

```
## Overnight Pass YYYY-MM-DD
### Summary
- Decayed: N entries
- Merged: N pairs (K flagged for review)
- Patterns: N surfaced
- Rehearsed: N promoted

### Patterns Observed
[...as above...]

### Contradictions
[...as above...]

### Rising Themes
[...as above...]
```

---

## What NOT To Do

This pass is about compression and pattern detection. It is NOT for:
- Writing new project status updates (daily pass).
- Processing today's debrief (daily pass + post-call-debrief skill).
- Editing Notion (daily pass + various skills).
- Surfacing morning brief content directly (that's begin-the-day's job... just leave patterns in overnight-log.md and let it pick them up).

If you find yourself wanting to do any of the above, stop. That's the wrong pass.
