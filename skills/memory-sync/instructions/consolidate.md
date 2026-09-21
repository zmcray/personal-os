# Phase 3: Consolidate

**Purpose:** Rewrite active-memory.md with everything learned. This is the living memory file that supplements CLAUDE.md with recent, dated, contextual knowledge.

---

## Steps

1. **Load existing active-memory.md** (from orient phase). If first run, start from the template in `templates/active-memory-template.md`.

2. **Merge new signal into the existing document.** For each piece of gathered signal:
   - If it updates an existing entry, replace the old entry with the new one (update the date)
   - If it's new, add it to the appropriate section
   - If it contradicts an existing entry, replace the old entry and add a note: "[Updated YYYY-MM-DD, previously: brief description of old state]"

3. **Date every entry.** Use absolute dates in YYYY-MM-DD format. Examples:
   - GOOD: "2026-04-02 [w:3] [src: session-2026-04-02]: Zack prefers call prep docs under 2 pages"
   - BAD: "Recently, Zack mentioned he likes shorter call prep docs"
   - BAD: "Last week, a new contact was added"

4. **Tag every new entry with weight and provenance.** Full schema lives at the top of active-memory.md. Every new entry follows this format:
   ```
   YYYY-MM-DD [w:N] [src: <tag>]: entry text.
   ```
   - Choose weight by importance. Default `w:3`. Bump to `w:4` for critical path / blockers. Use `w:5` only for thesis anchors and identity. Use `w:2` for background, `w:1` for ephemeral.
   - Choose a provenance tag that points to the source. Common tags: `debrief-<contact>-<date>`, `session-<date>-<topic>`, `granola-<date>`, `flag-that-<date>`, `email-<date>-<contact>`. Fall back to `manual` only when no source exists.
   - Backfill is opportunistic. Legacy entries without tags stay readable. Only tag them when you touch them.

5. **Write the updated file** to `40_OS/08_Memory/active-memory.md`.

6. **Write back durable observations to Supabase (added 2026-07-22).** When the pass identifies durable, recall-worthy facts (a decision made, a pattern observed, a state change future sessions will want to find semantically), write them to the unified memory store `os.memories` through `os.memory_write` via the Supabase MCP `execute_sql` (project `mtypgfwcebzsdlbuojef`). Changed 2026-09-21: `os.memory_observations` is deprecated; never insert into it directly.
   ```sql
   select os.memory_write(
     p_text    => '<one self-contained sentence or two>',
     p_kind    => 'observation',   -- or: thesis, idea, preference, decision, learning, reference, unknown
     p_tags    => array['<tag>'],  -- optional, lowercase words; omit with null
     p_topic   => null,            -- optional topic slug; unmatched names are kept as a hint, never block
     p_context => null,            -- optional one-line context
     p_source  => 'cowork',
     -- the function has no defaults: pass every remaining argument explicitly
     p_supersedes => null, p_capture_id => null, p_capture_table => null,
     p_capture_span => null, p_max_len => 2000
   );
   ```
   - **Pick the closest kind.** Use `decision` for ratified choices, `preference` for how Zack likes things, `learning` for lessons, `observation` for patterns. Use `unknown` rather than guessing.
   - The function dedupes identical writes within 60 seconds and returns a JSON envelope; `deduped: true` means the row already existed, which is fine.
   - **Calibration: 0 to 5 per day.** Durable facts and patterns, not routine status. "Email triage ran" does not qualify; "Zack ratified X approach for Y because Z" does.
   - **Each row must stand alone** without conversation context... it will be retrieved alone by vector search.
   - Retrieval is full-text search; write plain text and move on.
   - **Single-writer discipline unchanged:** only memory-sync writes `active-memory.md`. The memory store is an input and an output of memory-sync, never a bypass around it.

---

## Section Guide

Follow the template structure. Each section serves a specific purpose:

- **Active Context:** What's top of mind right now. Current projects, open threads, upcoming deadlines. This section changes the most.
- **Recent Corrections:** Things Zack explicitly corrected in the last 1-2 weeks. High retention priority.
- **People Updates:** New contacts, updated context on known people, relationship developments.
- **Project Status:** Where each active project stands. Brief, factual.
- **Patterns Observed:** Recurring behaviors, preferences, or themes noticed across sessions. These are hypotheses until confirmed 3+ times.
- **System Changes:** Skills created/updated, scheduled tasks changed, workflow adjustments.
- **Stale Flags:** Items from orient phase that need attention or confirmation.

---

## Consolidation Rules

- **Corrections beat observations.** If Zack said "do X", that overrides any inferred pattern.
- **Recency matters.** More recent signal gets more detail. Older entries get compressed.
- **Don't duplicate what's already in CLAUDE.md.** If CLAUDE.md already documents a preference or fact, don't repeat it in active-memory.md. Only add what's new, changed, or supplementary.
- **Don't duplicate what's in people profiles.** If a person has a full profile in `40_OS/08_Memory/people/`, active-memory.md should only note recent changes, not duplicate the profile.
- **Flag contradictions, don't resolve them.** If today's signal contradicts CLAUDE.md, note it in the Stale Flags section. Don't silently edit CLAUDE.md.

---

## McRayGroup.md Cross-Reference

During consolidation, check whether any new signal relates to the company's status, service offering, priorities, key relationships, active projects, pipeline, or open questions. These map to sections in `00_Context/McRayGroup.md`.

If signal would update McRayGroup.md, add a Stale Flag entry:
```
YYYY-MM-DD: McRayGroup.md update needed... [brief description of what changed]. Section: [section name]. Suggested action: update 00_Context/McRayGroup.md during next Strategy block.
```

Examples of McRayGroup.md-relevant signal:
- New client conversation or pipeline movement
- Service offering gets more specific (pricing, deliverables, engagement model)
- Priority shift or new blocker identified
- New key relationship that matters to the company
- Project status change on a company-related project
- Open question gets resolved

## Consolidate should NOT:
- Edit CLAUDE.md
- Edit any 00_Context files (including McRayGroup.md... flag changes, don't make them)
- Edit people profiles (that's a separate people-directory concern, handled ad hoc)
- Remove entries yet (that's prune)
