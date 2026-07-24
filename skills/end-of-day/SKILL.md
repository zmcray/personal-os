---
name: end-of-day
description: >
  Daily end-of-day wrap-up that closes out Zack's workday cleanly. Use this skill whenever the user says "wrap up", "end of day", "EOD", "close out the day", "daily wrap", or when the scheduled 4:45 PM ET wrap-up fires. Covers six things: (0) runs the day-ledger sweep across the day's Cowork session transcripts (decisions, work produced, open threads, next steps), (1) checks for any calls/meetings that still need debriefing, (2) gives a tomorrow prep summary with calendar and priorities, (3) closes the Roadmap loop on today's In progress tasks (mark Done or roll forward), informed by the ledger's completion evidence, (4) asks Zack to flag tomorrow's deep work focus, and (5) invokes the advisor skill for the evening chief-of-staff pass (tomorrow reassessment, uncertainty ranking, capped overnight research). Trigger this even if the user just casually says something like "anything I'm forgetting today?"
---

# End-of-Day Wrap-Up

## Model Policy (2026-07-05)
Scheduled runs: orchestrate at Sonnet. The day-ledger layer follows day-ledger's Haiku fan-out policy.

You are running Zack McRay's daily wrap-up. A quick, structured close-out that makes sure nothing falls through the cracks and tomorrow starts clean. Should feel like a 2-minute checkout, not a report.

**Tone:** Direct, actionable, no fluff. Always communicate in ET timezone.

---

## Workflow

### 0. Session Ledger (Part 0)
Invoke the **day-ledger** skill first. It sweeps all Cowork session transcripts since the last ledger, extracts decisions, work produced, open threads, and next steps, and writes the daily ledger to `40_OS/08_Memory/daily-ledger/`. Two handoffs back into this skill: (a) the ledger's completed-work evidence pre-fills Part 3's Done proposals so Zack confirms instead of recalls, and (b) the ledger's Next Steps queue is offered for Roadmap push inside the same Part 3 question, not as a separate ask. If day-ledger already ran today (file exists with today's date and no new sessions), skip with one line.

### 1. Debrief Sweep + Transcript Archive (Part 1, redesigned 2026-07-05)
Replaces the old same-day Debrief Catch, which let calls fall through permanently (in early testing, one call went 24 days undebriefed). Headless, 7-day lookback:
1. Pull Granola meetings for the last 7 days.
2. **Transcript archive:** any meeting without a file at `40_OS/08_Memory/call-transcripts/YYYY/YYYY-MM-DD-slug.md` gets archived (metadata + AI Summary + full verbatim transcript). Granola retention is ~30 days; this archive is the permanent record. Delegate to haiku subagents.
3. **Debrief check:** for each external-counterparty call (skip solo notes, webinars, family calls), check Attio for a debrief note on the person dated on/after the call.
4. Missing debrief → run it headless via the post-call-debrief synthesis format, written as a full Attio note on the person record + Attio follow-up tasks for Zack's commitments. Reference the transcript archive path in the note. Warranted follow-up emails go to Gmail as drafts (Voice Audit applies), never sent.
5. **Attio is the sole system of record for debriefs (decision 2026-07-05). Do not write to the Notion Meeting Notes database.**
6. **Draft-sent audit (loop-triage item 13, added 2026-07-05):** one Gmail call (`list_drafts`, metadata view) per run. Flag any draft older than 48 hours with recipient and age, plus the disposition prompt: "send, rewrite, or kill?" Zero stale drafts = silent, no output line. This closes drafted-never-sent... system skills (post-call-debrief, email-draft, thankyou, gmail-reply-drafter) create drafts but nothing sent them (April loop-log failure mode 2; 7 drafts accumulated Mar-Jun undetected). Details in `instructions/parts.md` Part 1.5.
7. Report in the wrap-up: transcripts archived, debriefs written, tasks created, drafts saved, stale drafts flagged (if any). Surface only calls Granola missed entirely for Zack's input. If all clear, one line and move on.

### 2. Tomorrow Preview (Part 2)
See `instructions/parts.md`. Pull tomorrow's calendar. Check CRM for context on external calls. Check if prep docs exist for tomorrow's calls. Surface overdue follow-ups. Show schedule highlights and flags.

### 3. Roadmap Completion Check (Part 3)
See `instructions/parts.md`. Query Supabase `os.roadmap_items` for today's in_progress items. Present them as a scannable checklist. Ask Zack which are done. For completed: status = 'done', completed_on = today, paired status_change event (actor zack). For incomplete: scheduled_for = tomorrow, paired reschedule event (actor zack); reschedule_count is trigger-maintained, flag items already at 3+. This is the bookend to begin-the-day Layer 0.

### 4. EOD Check-In (Part 4, expanded 2026-07-05)
See `instructions/parts.md`. One combined question, 15 seconds to answer, three fields: (a) "What should your Deep Work block be tomorrow?" (b) "What dragged... biggest friction?" (c) "Anything personal shaping tomorrow?" (travel, family, appointments... optional). Write to `40_OS/08_Memory/tomorrow-focus.md` (Focus / Friction / Constraints fields + timestamp). Energy is asked in the MORNING by begin-the-day, where the live read can shape the day... not here. This is the ONLY channel where Zack's internal state enters the system... the advisor and begin-the-day both read it, and no transcript contains it. Partial answers are fine; never nag for the optional fields. On unattended runs, write "Not flagged (unattended)" and move on.

### 5. Advisor Pass (Part 5, added 2026-07-05)
Invoke the **advisor** skill after Part 4 resolves (or times out on unattended runs... the advisor runs either way). It reads the ledger's facts and produces judgment: plan-vs-reality delta, uncertainty ranking, a direct call on tomorrow (engaging with Zack's Part 4 flag when one exists), and capped overnight research dispatch (max 2 subagents). It writes `evening-brief.md` + `morning-research.md` for begin-the-day. Headless: no questions, no chat output beyond one closing line ("Advisor brief written; [N] research agents running overnight."). Skip with one line only if no ledger exists AND the plan is missing.

---

## Output Format
See `templates/chat-output.md` for the exact template. Parts 1-2 in one chat message. Part 3 (Completion Check) as interactive question with scannable checklist. Part 4 (Flag Tomorrow's Focus) as a closing question after Part 3 resolves.

---

## Quality Checks
Before delivering, run through `eval/checklist.md`. Then imagine it being reviewed by the personas in `eval/advisory-board.md` to catch gaps.

---

## Common Failure Modes
See `examples/bad/anti-patterns.md` for what not to do. Most common: treating routine blocks as action items, skipping the tomorrow-focus question, asking for approval on Granola-captured debriefs instead of auto-processing.

---

## Reference Docs
- `00_Context/os-config.md` ... canonical config (database IDs, emails, paths, rules)
- `00_Context/working-rules.md` ... working preferences (Friday rule, evening overflow, deep work mornings)
- `shared/formatting-rules.md` ... no em dashes, timezone, tone
- `shared/calendar-rules.md` ... event classification, block architecture
- `shared/notion-databases.md` ... CRM ID
- `shared/folder-paths.md` ... all local folder paths
- `shared/crm-lookup.md` ... CRM lookup procedure
- `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md` ... canonical Supabase Roadmap schema, status model, and standard queries. Source for Part 3.

---

## Key Rules
- No em dashes anywhere. Use commas, periods, semicolons, or "..." instead.
- Keep it scannable. One scroll max.
- If a section has nothing to report, one line. No padding.
- Actually check Calendar, Drive, Notion. Don't guess.
- Hand off call debriefs to post-call-debrief skill, not inline.
- Preserve color coding when updating calendar events.
- On Thursday EOD, gently remind about Friday Weekly Review block.
- **Auto-process Granola-captured debriefs.** Don't ask for approval on these. Just do them.
- **Always ask the tomorrow-focus question.** This is the feedback loop that makes the morning launch work. Don't skip it.
