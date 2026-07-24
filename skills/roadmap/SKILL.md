---
name: roadmap
description: >
  Universal quick capture to the Roadmap. Use whenever the user says "/roadmap" followed by any text. Also trigger on "/task", "add to the backlog", "backlog this", "put this on the roadmap", "quick task", "add a task", "to-do", "todo", "remind me to", "I need to", or any phrasing where the user wants to capture actionable production work for later. Capture is intent-based: a slash command is an optional shortcut, not a requirement... any natural-language task intent gets captured without breaking flow. Does NOT cover builds ("/buildnote" routes to Linear) or relationship touches (follow up / schedule call / send note route to Attio, not here). Writes to Supabase (os.roadmap_items + os.roadmap_events), auto-classifies, confirms in one line.
---

# Roadmap

Universal quick capture for actionable production work: tasks, docs, decisions, research, builds-in-progress, sessions. One entry point, one table, auto-classified. Feeds the daily execution loop (Pulse, begin-the-day, end-of-day) and the weekly planning loop (Strategy sessions, weekly-review).

**This skill replaces `/task`.** That trigger still works as an alias (see Alias Routing below). **`/buildnote` is no longer an alias of `/roadmap`** ... as of 2026-04-28 it routes to Linear via the `buildnote` skill. Do not capture builds here.

**Conversational capture:** the slash command is optional. Any natural-language signal that Zack wants to do or remember something ("I need to...", "don't let me forget to...", "let's get that on the list") should be captured the same way, with the same one-line confirm. Do not wait for `/roadmap` or `/task` to be typed literally.

See `gotchas.md` for known failure patterns.

---

**Canonical schema:** `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md` (v1, 2026-07-21). This is the single source of truth for tables, columns, status model, and writer rules. Read it before writing any SQL if anything below feels stale.

## The Status Model (execution queue)

| Status | Meaning |
|--------|---------|
| **queued** | Committed. Has a `scheduled_for` date, or is explicitly next up. Surfaces daily. |
| **in_progress** | Actively being worked. WIP guard: flag when count > 3. |
| **later** | Real work, no commitment yet. Never in daily views. Surfaces only at Friday CONFIRM and when stale (`updated_at` 30+ days old). |
| **done** / **dropped** | Terminal. `done` sets `completed_on`. |

There is **no idea/backlog status**. Speculative or exploratory items go to the vault, Linear, or a thinking file, not the Roadmap.

**The flow:**

1. Capture lands as `queued` if a date/when is present, else `later` (default, no time signal).
2. Strategy session or weekly review promotes `later` → `queued` and assigns `scheduled_for`.
3. Direct today-capture (`/task`, or `/roadmap`/conversational capture with "today"/"now" signals) lands `queued` with `scheduled_for` = today.
4. When Zack picks up a `queued` item to work on, it moves to `in_progress`.
5. When finished, it moves to `done` with `completed_on` set.

---

## Boundary Rule: Roadmap vs. Attio (decision 2026-07-21)

- **Attio owns relationship touches.** "Follow up with X", "schedule a call with X", "send X a note", cadence pings → create an Attio task on the person record. Do NOT put these on the Roadmap.
- **Roadmap owns production.** Docs, decisions, research, builds, sessions, internal tooling, anything that isn't a relationship touch.
- No sync between the two systems. Read surfaces (Pulse, begin-the-day) merge both when relevant.
- If a capture is genuinely ambiguous (e.g., "prep and send Jordan the deck"), lean Roadmap... it's production work with a relationship byproduct, not a pure touch.

---

## Alias Routing

| Trigger | Behavior |
|---------|----------|
| `/roadmap [text]` | Full classification. Default status = `later` (no time signal) or `queued` (time signal present). |
| `/task [text]` | Task-routing heuristic (see Detect Type below). Default status = `queued`, `scheduled_for` = today (unless text signals otherwise). |
| Conversational capture (no slash command) | Same logic as `/roadmap`; infer today/queued vs. later from the text. |
| `/buildnote [text]` | **Routed elsewhere.** Buildnotes go to Linear via the dedicated `buildnote` skill, NOT to the Roadmap. Do not handle here. |
| Relationship touch (follow up / schedule call / send note) | **Routed elsewhere.** Create an Attio task, not a Roadmap row. |

---

## How It Works

### 1. Extract the Item
Everything after the trigger prefix (or the natural-language task intent) is the item. Use it as-is. Do not rewrite, summarize, or edit. Strip only the trigger prefix ("/roadmap", "/task") and pure scheduling words used for the date.

### 2. Apply the Attio Boundary Check
Before writing anything, check: is this a relationship touch (follow up, schedule call, send note, cadence ping)? If yes, this skill does not apply... hand off to an Attio task instead. Otherwise, continue.

### 3. Detect Task vs. Idea (routing heuristic, not a field write)
There is no separate Type or Idea status. This step only decides priority/notes framing, not a status to write beyond the time-signal routing in Step 5.

| Signal patterns | Routing |
|----------------|------|
| Short action verb phrases: "email", "call", "send", "review", "update", "set up", "check", "submit", "book", "cancel"; build/create language ("build", "create", "tool", "app", "system", "automate", "prototype", "design", "develop", "ship", "integrate", "agent", "workflow", "dashboard", "template", "pipeline") | Production ... route per time-signal routing (Step 5) |
| Exploratory/strategic: "explore", "research", "think about", "what if", "consider", "look into", "evaluate", "test whether", "could we", or framed as a question/hypothesis | Still captured here (there is no idea status) ... default `later`, P3 unless otherwise signaled |

Rules:
- Default to production routing if ambiguous. Most captures are actionable.
- **Never ask which type.** No Type field exists.

### 4. Detect Workstream
Resolve against the 7 seeded `os.workstreams` slugs: `acquisitions` (primary), `consulting`, `product`, `gtm`, `network`, `os`, `career`. A cheap `select id, name from os.workstreams` (or working from the list above, if confidently memorized) is enough... do not run a full table scan for every capture.

Rules:
- If the item clearly maps to one of the 7 slugs, set `workstream_id`.
- If genuinely unclear, leave `workstream_id` null. **Never ask which workstream**, and never guess to force a match. A wrong workstream is worse than null.

### 5. Detect Scheduling (Smart Routing)
Time signals determine both `scheduled_for` and `status`.

| Signal | scheduled_for | status |
|--------|--------------|--------|
| "today", "now", "this afternoon" | today | `queued` |
| "tomorrow" | tomorrow | `queued` |
| "this week" | today | `queued` |
| "next week", "next month", "eventually", "someday" | null | `later` |
| No time signal + `/task` alias | today | `queued` |
| No time signal + `/roadmap` or conversational capture | null | `later` |

The `/task` alias defaults to "capture as today's work" because the user is signaling "I want to do this now." `/roadmap` and plain conversational capture default to `later` because there's no explicit commitment yet.

### 6. Detect Priority
Default `P3`. Only override if the text signals urgency (P1/P2, e.g. "urgent", "ASAP", "ship today") or explicit deprioritization (P4, e.g. "someday", "no rush", exploratory framing).

### 7. Detect Source Skill Context
Check the current conversation context. If the user was actively using another skill (begin-the-day, end-of-day, weekly-review, signal-session, etc.), note it in `notes` as "Captured during: [skill-name]". If no skill is active, leave `notes` blank.

### 8. Write to Supabase
Use the Supabase MCP `execute_sql` tool (name ends in `__execute_sql`) against `project_id = <SUPABASE_PROJECT_ID>`. Run the insert and its matching event **in the same SQL batch**:

```sql
with new_item as (
  insert into os.roadmap_items (item, status, workstream_id, priority, scheduled_for, source, notes)
  values ($item, $status, $workstream_id, $priority, $scheduled_for, 'ad-hoc', $notes)
  returning id
)
insert into os.roadmap_events (item_id, event, actor, detail)
select id, 'created', 'cowork', jsonb_build_object('via', 'roadmap-skill')
from new_item
returning item_id;
```

- `item`: the captured text (strip trigger prefix and pure scheduling words; e.g., "/task email Casey tomorrow" becomes "Email Casey" with `scheduled_for` = tomorrow).
- `status`: smart-routed (`queued` or `later`; never `idea`).
- `workstream_id`: resolved slug or null.
- `priority`: `P3` default unless signaled otherwise.
- `source`: always `'ad-hoc'`.
- `notes`: source skill context if applicable, otherwise null.
- There is no objective, effort, project, parent, linear_issue_url, or calendar_event_id column. Do not reference them.

### 9. Confirm in One Line
Format: `Roadmap: "[item]" > [status] [priority] [workstream if set] [date if applicable]`

Examples:

- `/roadmap build a tool that ingests company financials for client engagement`
  → `Roadmap: "Build a tool that ingests company financials for client engagement" > later [P3] [consulting]`
- `/task email Bob about the portco deck`
  → `Roadmap: "Email Bob about the portco deck" > queued [P3] [network] today`
- `/roadmap explore white-labeling the reporting layer for clients`
  → `Roadmap: "Explore white-labeling the reporting layer for clients" > later [P3]`
- `oh I need to call Alex tomorrow` (conversational capture, no slash command)
  → `Roadmap: "Call Alex" > queued [P3] tomorrow`

That is it. No follow-up questions. No suggestions. Return the user to what they were doing.

---

## Reference Docs

- `40_OS/10_Migrations/2026-07-roadmap-supabase/supabase-roadmap-contract.md` ... canonical Supabase schema, status model, writer map
- `40_OS/08_Memory/workstreams.md` ... workstream reference (big rocks); cross-check against the 7 seeded slugs

---

## Rules

- **Never ask a follow-up question.** Capture and confirm.
- **Never rewrite the item.** Typos, fragments, shorthand are fine. Capture exactly.
- **Source is always `ad-hoc`.**
- **Keep confirmation under 20 words.**
- **Workstream is best-effort.** Leave null unless a slug clearly matches. Wrong workstream is worse than null.
- **Status defaults: `/roadmap` and conversational capture = `later` unless a time signal is present. `/task` = `queued` today.** `/buildnote` is handled by the buildnote skill (Linear), not here.
- **Every insert to `os.roadmap_items` gets a matching `created` event in `os.roadmap_events` in the same batch.** Actor = `cowork`.
- **Relationship touches go to Attio, not here.** See Boundary Rule above.
- **Respect alias routing.** Do not override alias-forced status unless the text contains explicit overrides.

---

## How This Fits the System

**This is the capture point for actionable production work.** It replaces `/task`. Builds are NOT captured here ... `/buildnote` routes to Linear. Relationship touches are NOT captured here ... they route to Attio.

Items surface in:
- **Pulse**: Reads `os.roadmap_items` directly (read-only). Today view shows `in_progress` + `queued` scheduled today.
- **Begin the Day** (Layer 2: Priority Stack): reads today view, may promote scheduled items.
- **End-of-Day** (Part 4: Completion Check): approval-gated update of status/completed_on on today's `in_progress` items.
- **Strategy sessions** (Tue/Thu): Triage `later` items. Promote to `queued` + assign `scheduled_for` + set workstream.
- **Weekly Review CONFIRM** (Friday, attended): promotes `later` → `queued`, kills stale items.
- **Weekly Review RETRO, advisor, pulse**: read-only.
- **Memory sync** (nightly): Patterns across captures surface into active memory.
