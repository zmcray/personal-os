# Supabase Roadmap Contract (v1, 2026-07-21)

Canonical reference for every skill that reads or writes the Roadmap. Replaces all Notion Roadmap logic. The Notion Roadmap and Workstreams DBs are FROZEN (ARCHIVED prefix, read-only, 2-week fallback, delete only with Zack's explicit approval).

## Connection
- Supabase MCP `execute_sql` tool (server prefix varies by session; find the tool whose name ends in `__execute_sql`).
- `project_id`: `<SUPABASE_PROJECT_ID>` (project `<SUPABASE_PROJECT_NAME>`).
- Schema: `os`. RLS on, no policies; MCP/service-role access only.

## Tables
### os.workstreams
`id` (slug pk) · `name` · `description` · `status` (active/paused/retired) · `sort_order`
Seeded slugs: `acquisitions` (primary), `consulting`, `product`, `gtm`, `network`, `os`, `career`.

### os.roadmap_items
`id` uuid pk · `item` · `status` · `workstream_id` fk · `priority` (P1-P4, default P3) · `scheduled_for` date · `due_date` date · `completed_on` date · `source` (default 'ad-hoc') · `notes` · `reschedule_count` (TRIGGER-maintained... never set manually; increments when scheduled_for moves later) · `notion_id` (provenance) · `created_at` / `updated_at` (trigger).
There is NO objective, effort, project, parent, linear_issue_url, or calendar_event_id column. Do not reference them.

### os.roadmap_events (append-only)
`item_id` fk · `event` (created / status_change / reschedule / priority_change / edit) · `actor` (cowork / hermes / zack) · `detail` jsonb · `at`.
**EVERY write to roadmap_items MUST insert a matching event in the same SQL batch.** Actor: `cowork` for loop/skill-driven writes, `zack` for Zack's approval-gated decisions.

## Status model (execution queue)
- `queued`: committed. Has a `scheduled_for` date or is explicitly next up. Surfaces daily.
- `in_progress`: actively being worked. WIP guard: flag when count > 3.
- `later`: real work, no commitment. NEVER in daily views. Surfaces only at Friday CONFIRM and when stale (updated_at 30+ days old → promote-or-kill).
- `done` (set `completed_on`) / `dropped`: terminal.
- There is NO idea/backlog status. Ideas go to the vault, Linear, or thinking files... not the Roadmap.

## Capture rules
- Capture is INTENT-BASED. Slash commands are optional shortcuts; any natural-language task intent ("I need to...", "don't let me forget...", voice) gets captured without breaking flow.
- Insert: status = `queued` if a when exists, else `later`. Priority default P3. Workstream resolved from the 7 slugs (1 cheap select or from memory of the list above); nullable if unclear... never ask more than one short question, never block capture.
- Boundary rule (decision 2026-07-21): **Attio owns relationship touches** (follow up, schedule call, send note, cadence ping → Attio task on the person record). **Roadmap owns production** (docs, decisions, research, builds, sessions). No sync between them; read surfaces merge both.

## Writer map
| Actor | Access |
|---|---|
| Capture (roadmap/task skills, conversational intent), post-call-debrief | INSERT (debrief inserts production work only; touches → Attio) |
| day-ledger / end-of-day (approval-gated) | UPDATE status/completed_on, INSERT next steps |
| begin-the-day | READ today view; UPDATE promote scheduled items |
| weekly-review CONFIRM (attended) | UPDATE promote later→queued, kill stale |
| weekly-review RETRO, advisor, pulse | READ only |
| Pulse Plan tab, Cowork artifact `mcray-os-roadmap` | READ only |
| Pulse Daily tab (added 2026-07-22, MCR-<N>) | UPDATE status/notes/scheduled_for via `pulse_set_task_status` / `pulse_set_task_notes` / `pulse_reschedule_task` RPCs only (approval-gated: every write is a direct authenticated Zack action behind Basic Auth; actor `zack`; each RPC pairs the update with its typed event in one statement; no-ops write nothing) |
| Hermes (future) | rows only, approval-gated, never DDL |

## Standard queries
- Today view: `status='in_progress' OR (status='queued' AND (scheduled_for <= current_date OR due_date <= current_date))`
- Week view: queued/in_progress ordered by scheduled_for.
- Stale later: `status='later' AND updated_at < now() - interval '30 days'`
- Reschedule flag: `reschedule_count >= 3` (begin-the-day names these).
- Priority calibration (Friday RETRO): completion order vs priority for the week from roadmap_events.
- Mark done: `update os.roadmap_items set status='done', completed_on=current_date where id=...;` + status_change event.

## Gone (do not reference)
- Notion Roadmap collection `<NOTION_COLLECTION_ID_ROADMAP>` and Workstreams `<NOTION_COLLECTION_ID_WORKSTREAMS>` (frozen).
- The Notion plan-gate fetch+filter workaround. Real SQL now; delete those instruction blocks.
- Rollover Count manual increments (trigger handles reschedule_count).
- Objective tag, Type/Workblock/Domain, the 22-name workstream keyword table.
