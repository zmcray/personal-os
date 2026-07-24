---
name: day-ledger
description: >
  Sweep all Cowork session transcripts since the last ledger, extract decisions made, work produced (with file locations), steps completed, open items still mid-flight, and next steps, then synthesize a single daily ledger grouped by project. Use this skill whenever the user says "/ledger", "day ledger", "run the ledger", "what did I do today", "where did today go", "recap my day", "recap my sessions", "what's still open", "what did we decide today", "what needs my review", or any phrasing that signals they want a cross-session accounting of the day's work. Also triggered as a layer by the end-of-day skill. Writes the ledger to 40_OS/08_Memory/daily-ledger/ and runs a two-way Roadmap loop (Supabase-backed): marks completed Roadmap tasks done and pushes approved next steps in. Do NOT trigger for single-meeting debriefs (use post-call-debrief), Roadmap-only checkout (use end-of-day), or memory consolidation (use memory-sync).
---

# Day Ledger

## Model Policy (2026-07-05)
Delegate transcript mining to parallel subagents with model: haiku, one per session transcript. Synthesis of the ledger and the Roadmap loop run at the orchestrator (Sonnet for scheduled runs).

You are building Zack's daily ledger: a cross-session accounting of what was decided, produced, completed, and left open across every Cowork session since the last ledger ran. The pain this solves: work happens in many parallel sessions and the threads get lost. The ledger is the single place where the day's threads are tied off or explicitly carried forward.

**Tone:** Direct, scannable, ET timezone, no em dashes (use "..." instead).

---

## Workflow

### 1. Sweep (instructions/sweep.md)
Read the watermark from the latest ledger file, list recent sessions via `list_sessions`, identify unprocessed sessions, classify each as substantive or routine. Delegate transcript mining to background agents (one batch per 3-4 sessions) so the main context stays clean. Each agent returns a structured per-session digest.

### 2. Synthesize (instructions/synthesis.md)
Merge per-session digests into one ledger grouped by project. Sections: Decisions, Work Produced, Completed, Still Open / Mid-Flight, Needs Your Review, Next Steps, **Open Questions & Uncertainty** (added 2026-07-05: moments where Zack hedged, circled a decision without landing it, asked a question no session answered, or decided on visibly thin information... the miners capture these verbatim-adjacent with session pointers; the advisor skill reads this section every evening). Write the file to `40_OS/08_Memory/daily-ledger/YYYY-MM-DD.md` using `templates/ledger.md`. Include the session watermark block. After the Decisions section is final, route firm-level decisions to the McRay Group vault decision-notes file (see "Decision Routing" in instructions/synthesis.md) and tag them in the ledger as routed.

### 3. Roadmap Loop (instructions/roadmap-loop.md)
Two directions against Supabase (`os.roadmap_items` / `os.roadmap_events`), both approval-gated via one AskUserQuestion:
- **Close out:** cross-reference today's in_progress Roadmap items against completed work; propose marking them done.
- **Push in:** propose extracted next steps as new Roadmap items with priority.
Execute only what Zack approves. Every write pairs with a roadmap_events insert in the same SQL batch (actor `zack` on approval-gated writes).

**Headless catch-up mode (added 2026-07-05):** when invoked by the weekly-review RETRO (Friday ~11:45 AM) because EOD missed days, run steps 1-2 in full (sweep, synthesize, write ledger files, firm-decision routing) but SKIP the AskUserQuestion gate entirely. Return the close-out and push-in proposals to the caller instead... they land in the retro packet and get approved in the Friday CONFIRM session. Never write to the Roadmap in this mode. No chat delivery either; the caller owns the output.

### 4. Deliver
One scannable chat summary (one scroll max): headline counts, decisions, needs-review list, next steps queue, link to the ledger file via computer:// link. Then the approval question. Done.

---

## Key Rules
- **Watermark, not calendar.** Process sessions not covered by the last ledger. If no prior ledger exists, take the 15 most recent sessions and confirm scope with Zack before mining.
- **Delegate the reading.** Never read full transcripts in the main session. Background agents only. Main chat gets digests.
- **Routine sessions get one line.** Scheduled runs (memory-sync, crm-auto-check, signal-classify, active-projects-sync, begin-the-day, end-of-day, pulse refreshes) are noted under "Background ops", not mined.
- **Skip child sessions** (`is_child: true`) ... their work is already represented in the parent's transcript.
- **Skip the current session** (the one running this skill).
- **Every file claim needs a path.** "Work produced" entries must include the actual file location or DB link. If the agent can't find where output landed, say so explicitly under Needs Your Review.
- **Decisions are verbatim-faithful.** Capture what was actually decided, not a paraphrase that drifts. Faithful to substance, compressed in wording... do not paste transcript paragraphs. Include enough context to be unambiguous in 3 months.
- **Firm-level decisions route to the company vault.** McRay Group strategy, thesis, brand, capital, acquisitions, positioning decisions get appended to `01-mcray-group/50-vault/thinking/decision-notes.md` with a pointer back to the ledger. Personal admin and OS/tooling mechanics do not route. Routing feeds the weekly decision triage; it never promotes to `decisions/` directly.
- **Open items are the point.** A session that ended mid-thought is exactly what this skill exists to catch. Err toward listing it.
- **Uncertainty is a first-class capture, description-first.** Miners describe each material uncertainty in one sentence (quote optional, as evidence); materiality filter applies (project / target / money / strategy only, never stylistic hedges); ledger section caps at top 5 with a recurrence scan across the last 5 ledgers. Capture, don't interpret... the advisor does the ranking.
- **Never write to Notion without approval.** The AskUserQuestion gate covers both Done-marking and next-step pushes.
- **If a section is empty, one line.** No padding.

---

## Reference Docs
- `00_Context/os-config.md` ... canonical config (database IDs, emails, paths)
- `gotchas.md` ... known failure patterns. Read before first run.
- `shared/formatting-rules.md` ... no em dashes, timezone, tone

---

## How This Fits the System
- **end-of-day** invokes this as its session-ledger layer before the Roadmap completion check (the ledger's close-out proposals feed Part 3 directly).
- **begin-the-day** reads yesterday's ledger for morning context on open threads.
- **memory-sync** mines ledger files for patterns during the nightly pass.
- Standalone trigger works any time of day ("where did today go?").
