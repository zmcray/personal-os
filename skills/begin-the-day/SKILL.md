---
name: begin-the-day
description: >
  Morning launch sequence that boots up Zack's operating system for the day. Use this skill whenever the user says "start my day", "begin the day", "morning launch", "good morning", "what's my day look like", "boot up", or when the scheduled 6 AM ET auto-run fires. Also trigger on casual phrasing like "what do I have going on today?" or "let's get started". This replaces the standalone daily-call-prep skill by incorporating call prep as one layer of a fuller morning briefing. If the user only wants call prep for a specific meeting, use the daily-call-prep skill instead.
---

# Begin the Day

## Model Policy (2026-07-05)
Scheduled/headless runs: orchestrate at Sonnet. Delegate each call-prep brief to a parallel subagent (model: sonnet) via daily-call-prep. The strategic priority-recommendation layer may use one subagent (model: opus). Attended runs inherit the session model.

You are running Zack McRay's morning launch sequence. A unified daily briefing that answers "What should I focus on today?" in under 2 minutes of reading, with call prep docs generated as files for deeper reference.

**Tone:** Direct, actionable, no fluff. Think like a chief of staff briefing a principal. Always communicate in ET timezone.

---

## Workflow

### 0.1. System Heartbeat (Layer 0.1)
See `instructions/layers.md`. Runs FIRST, before Layer 0. Fast health check (≤60s, ≤5 tool calls) on the automation layer: (1) Cowork scheduled tasks... every ENABLED task's lastRunAt vs. its most recent expected cron fire in ET, 2-hour grace, weekend/weekly/monthly rules, cap 5 flags; (2) n8n... error/crashed executions in the last 24h plus a success-in-26h cross-check on expected-ACTIVE workflows from `00_Context/databases.md`; (3) file-freshness fallback (mtimes on sync-log, overnight-log, active-projects, latest daily ledger) if either tool is unavailable; (4) Argus mutual watchdog... one Supabase statement that upserts the Mac's `os.heartbeats` beat and flags if Argus's hourly beat is >24h stale (added Step 9, 2026-07-22). Output: one green line when healthy, or a "Heartbeat flags:" block (max 6 lines) with last-seen times and suggested actions. Replaces the retired schedule-watchdog (2026-07-05).

### 0. Task Promotion (Layer 0)
See `instructions/layers.md`. Before running any briefing layer, execute the mechanical task promotion step against Supabase (`os.roadmap_items` / `os.roadmap_events`, per `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md`): (a) pull the Today view (`in_progress` or `queued` with `scheduled_for`/`due_date` <= today), (b) promote due-today `queued` items to `in_progress` with a paired `status_change` event (actor `cowork`), (c) rot-guard surfacing: flag WIP > 3 in_progress items, name overdue items, name items with `reschedule_count >= 3`. Output is a "Task Prep" micro-section. If nothing needs attention: "All clear. Today's tasks already staged."

### 0.25. Capture Sync (Layer 0.25)
Run the `sync-captures` skill silently to flush any mobile/web captures from the Notion **Capture Inbox** (`collection://<NOTION_COLLECTION_ID_2>`) into the local vault thinking files. This must run BEFORE any later layer reads from `01-mcray-group/50-vault/thinking/` so the briefing reflects overnight captures. If captures were synced, surface a one-liner in the briefing: `Synced <N> mobile captures: thesis=<N>, strategy=<N>, brand=<N>, playbook=<N>.` If nothing was pending, suppress this line entirely. See `40_OS/05_Skills/sync-captures/SKILL.md` for full logic.

### 0.5. CRM Check Surface (Layer 0.5)
See `instructions/layers.md`. Reads the output of the crm-auto-check scheduled task (runs 5:45 AM ET). Surface layer only, never runs CRM scan logic itself. 3-4 lines or omitted entirely.

### 1. Read Today's Inputs (Layer 1)
See `instructions/layers.md`. Read four files: (a) `40_OS/08_Memory/tomorrow-focus.md` for what Zack flagged last night at EOD, (b) `40_OS/08_Memory/evening-brief.md` for the advisor's overnight recommendation (Tomorrow's Call, Week Delta, Watchout), (c) `40_OS/08_Memory/morning-research.md` for overnight research results to surface in the briefing, and (d) `40_OS/08_Memory/next-week-plan.md` for the week's priorities (priorities-only format as of 2026-07-04: Priority Workstreams, Weekly Targets, Terrain Block Focus, Open Afternoon Fallbacks, Status DRAFT/RATIFIED... no per-day block table). These feed Layers 2, 3, and 4. If Status is DRAFT on a Monday, flag once that the plan is unratified.

### 2. The Day at a Glance (Layer 2)
See `instructions/layers.md`. Pull today's calendar, characterize the shape of the day. 2-4 lines max. Skip list: Morning Launch, Lunch, Daily Wrap-Up, Terrain Block (except one short focus line when the plan sets a Terrain Block Focus). Everything else is narrated, including the deep work block with its focus. Flag any conflicts with the protected deep work window (8:45-10:45 AM Mon-Thu / 8:45 AM-12:00 PM Fri).

### 2.5. Targets This Week (Layer 2.5)
See `instructions/layers.md`. Reads `00_Context/active-projects.md` and surfaces any project with a Target Date within 7 days. One line per project. Conditional render ... if nothing's imminent, suppress entirely. Time-sensitive context fed BEFORE the deep work recommendation so Layer 3 can react to it.

### 3. Deep Work Recommendation (Layer 3)
See `instructions/layers.md`. The core recommendation. **Primary inputs:** (1) tomorrow-focus.md flag from EOD, (2) live assignment built from the weekly plan's Priority Workstreams + Weekly Targets (the plan has no per-day rows as of 2026-07-04... this layer picks today's workstream, project, and 1-3 tasks itself). Tomorrow-focus.md (if freshly set) overrides; the weekly priorities are the strategic lens. Cross-reference with PROJECT.md files for milestone deadline overrides. Only project-level work belongs here. Small tasks, follow-ups, and CRM items go to Layer 4. Output: "Your deep work block should be: [specific project + what to work on] because [reason]." **Workstream enrichment:** after the recommendation, append a single-line "Other [Workstream] options" list pulled from active-projects.md (cap 3, sorted by target proximity, excluding the recommended project).

### 4. Flexible Block Recommendation (Layer 4)
See `instructions/layers.md`. If there's open time beyond calls and the deep work block, suggest what to do with it. Pulls from: today's open afternoon fallback workstreams in next-week-plan.md, Roadmap tasks scheduled for today, CRM follow-ups, commitments from recent calls. If no open time and items are overdue or time-sensitive, flag them as evening candidates. If nothing is urgent, skip.

### 5. Call Prep (Layer 5)
See `instructions/layers.md`. Identify qualifying events, then DELEGATE to the **daily-call-prep** skill scoped to today (canonical home for all call-prep logic as of 2026-07-04... doc detection, freshness checks, Attio lookups, .docx generation all live there). In-chat summary shows one line per call: time, contact, company, and what to drive toward. Link to the prep doc.

### 6. Signal Feed Insights (Layer 6)
See `instructions/signal-feed.md` for how to pull and present signal items from the Signal Library. Lead with 2-3 items that connect to today's agenda. 5-8 lines max. If nothing new, skip.

### 7. Recommended Read (Layer 7)
See `instructions/layers.md`. If the day has a realistic 15-20 minute window, recommend ONE article or podcast with a link. Offer to schedule a calendar block for it. If Zack says yes, create the event with the article linked in the description. If no time available, skip.

---

## Output Format
See `templates/chat-output.md` for the exact template. Deliver everything in a single chat message. Keep it tight and scannable.

---

## Quality Checks
Before delivering the briefing, run through `eval/checklist.md` to confirm the output is solid. Then imagine it being reviewed by the personas in `eval/advisory-board.md` to spot any gaps.

---

## Common Failure Modes
See `examples/bad/anti-patterns.md` for what not to do. Most common: signal feed becoming a content dump, padding the priority stack when nothing is urgent, ignoring the tomorrow-focus.md flag, surfacing small tasks in the deep work recommendation instead of project-level work.

---

## Reference Docs
- `00_Context/os-config.md` ... canonical config (database IDs, emails, paths, rules, **Daily Shell** in Section 3)
- `00_Context/working-rules.md` ... working preferences (deep work mornings, calls afternoon, evening overflow rules)
- `00_Context/active-projects.md` ... live snapshot of all in-flight projects (refreshed daily 5 AM ET by `active-projects-sync`). Source for Layer 2.5 (Targets This Week) and Layer 3 workstream enrichment.
- `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md` ... canonical Supabase Roadmap schema, status model, and standard queries. Source for Layer 0.
- `40_OS/05_Skills/sync-captures/SKILL.md` ... drains the Notion Capture Inbox into the vault. Run silently in Layer 0.25 before vault reads.
- `40_OS/08_Memory/next-week-plan.md` ... weekly plan drafted headlessly by the Friday retro, ratified in the confirm session. Priorities only; Layer 3 assigns today's block live.
- `40_OS/08_Memory/tomorrow-focus.md` ... last-night EOD flag, freshest signal
- `shared/formatting-rules.md` ... no em dashes, timezone, tone
- `shared/calendar-rules.md` ... event classification, energy management
- `shared/notion-databases.md` ... Notion database IDs (CRM section points to Attio)
- `shared/folder-paths.md` ... all local folder paths
- `shared/crm-lookup.md` ... Attio lookup procedure (CRM is Attio, migrated from Notion mid-2026)
- `shared/crm-schema.md` ... valid Attio attribute values (contact_type, contact_cadence, auto-computed interaction fields, tasks, notes)

---

## Key Rules
- **Heartbeat runs first and never blocks... one green line or a capped flag block.** If the heartbeat itself errors, say so in one line and continue the briefing.
- **Clean output only.** The first line of output must be "Good Morning ..." No reasoning, no intermediate steps, no tool call narration, no progress tracking, no todo lists. Just the briefing.
- **Morning energy ask (2026-07-05):** the briefing closes with "Energy this morning, one word?" A low answer swaps the deep work call to the advisor brief's low-energy alternative. Deliver first, ask last, never block on it.
- No em dashes anywhere. Use commas, periods, semicolons, or "..." instead.
- Chat summary should be scannable in 60 seconds. Detail lives in prep docs.
- Actually query Calendar, Attio, Drive, and Signal Library. Don't guess.
- If a call attendee has no Attio record, flag it.
- Signal feed section earns its place by being useful to today, not by being a content dump.
- Call prep docs use bold-lead bullet format, not headers-and-paragraphs.
- **Protected deep work block:** Never suggest scheduling calls during 8:45-10:45 AM Mon-Thu or 8:45 AM-12:00 PM Friday. Flag any conflicts.
- **Input priority for deep work recommendation:** (1) tomorrow-focus.md if freshly set last night (Zack's word; if the advisor brief challenged it, surface the counter-case in one line), (2) the advisor brief's "Tomorrow's Call", (3) live assignment built from the weekly plan's priorities, (4) PROJECT.md deadline override if a milestone is within 3 days.
- **Deep work recommendation is project-level only.** Small Roadmap tasks, CRM follow-ups, and one-off to-dos belong in the Flexible Block Recommendation, not here.
