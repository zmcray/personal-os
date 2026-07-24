---
name: signal-classify-sync
description: Classify new signal items from Notion and sync classified items to Obsidian vault
schedule: daily, 7:00 AM ET
---

# Signal Classify + Sync

Daily classification and vault sync for Zack's signal feed. Run every morning at 7 AM ET. Sources: Signal Library (single database covering tweets via n8n Readwise-to-Notion sync, articles via web clipper, and podcast clips from Snipd).

## Workflow

### Step 1: Read Gotchas
Before starting, read `gotchas.md`. Known failure patterns live there. Proceed only after reviewing.

### Step 2: Source Unclassified Items
Read `instructions/sourcing.md` for search strategy and data source details.
Read `references/readwise-schema.md` for the complete database schema.

Run 6+ search queries across the Signal Library (`collection://<NOTION_COLLECTION_ID>`) to find all items with empty Signal Type. This single database contains tweets (via n8n Readwise-to-Notion sync), articles (from web clipper), and podcast clips (from Snipd).

If a Snipd Notion database ID is configured in `references/snipd-db.md`, run the same queries against that database and merge results. Dedupe by URL across both sources.

For each unclassified item, fetch full page content (title, author, highlights, body, Last Highlighted date).

Log count of items found.

### Step 3: Classify
Read `instructions/classification.md` for the locked 3-bucket decision tree (Signal / Reference / Learn) plus the substance-over-titling rule, the tool-mention rule, the calibration examples, and the truncated-preview fetch logic (Option B).

For each unclassified item:
1. Read the full content (not just title)
2. Walk the decision tree: Learn (build/practice intent)? else Reference (citeable artifact)? else Signal (absorbing pattern)? else leave unclassified.
3. If borderline AND preview is truncated: fetch full tweet via Chrome MCP, then re-classify.
4. Record the decision.

Log counts: Classified as Signal / Classified as Reference / Classified as Learn / Left for review.

### Step 4: Update Notion
For each classified item, use `notion-update-page` to set the "Signal Type" property:
- "Signal Type": "Signal" for signal items
- "Signal Type": "Reference" for reference items
- "Signal Type": "Learn" for learn items

Do NOT set "Action" automatically. Action items are manual-review only. The "Action" option remains in the property for legacy reasons.

### Step 5: Sync to Vault
Read `instructions/vault-sync.md` for sync rules and file handling.
Read `references/vault-conventions.md` for Obsidian formatting conventions.
Use templates from `templates/signal-note.md`, `templates/reference-note.md`, and `templates/learn-note.md`.

For each classified item:
1. Generate filename: YYYY-MM-DD-slug-from-title.md (use Last Highlighted date, not created date)
2. Check if file already exists in target folder (Signal/ for signals, Reference/ for references)
3. If exists: skip and log as "already in vault"
4. If not exists: create file with appropriate template, fill frontmatter and content
5. Strip all em dashes from content (replace with "..." or commas)

Log counts: Synced to vault / Skipped (already existed).

### Step 6: Report
Read `instructions/reporting.md` for report format and structure.

Generate brief summary:
- ET timestamp (YYYY-MM-DD HH:MM ET)
- Sources queried (Readwise Library + Snipd DB if active)
- Counts: found unclassified, classified as Signal/Reference, left for review
- Sync counts: synced to vault, skipped
- Noteworthy items: bare links (probable X Articles), ambiguous cases, interesting signals
- Any errors or blockers

### Step 7: Verify
Run `eval/checklist.md` to verify all steps completed correctly before marking task done.

## Rules

- No em dashes anywhere in generated files. Use commas, periods, semicolons, or "..." instead.
- Use ET timestamps always.
- Never auto-set "Action" for Signal Type. That's manual review only.
- Tie-breaker: Learn > Reference > Signal. Learn wins when there's a buildable method even if a tool is named.
- Substance over titling. A catchy named phrase doesn't auto-route to Reference if the body is observational.
- When in doubt on classification, leave unclassified. Zack will triage.
- Do not overwrite existing vault files. Skip if one already exists.
- Leave comment sections (<!-- -->) empty in vault notes for Zack to fill during manual review.

## Key Files

| File | Purpose |
|------|---------|
| gotchas.md | Known failure patterns, read first |
| instructions/sourcing.md | How to find unclassified items (Readwise Library + optional Snipd DB) |
| instructions/classification.md | 3-bucket decision tree (Signal / Reference / Learn) with calibration examples and Option B fetch logic |
| instructions/vault-sync.md | Vault sync and file handling logic |
| instructions/reporting.md | Report format and content |
| references/readwise-schema.md | Database schema and properties |
| references/snipd-db.md | Snipd Notion DB config (activate when Snipd integration is live) |
| references/vault-conventions.md | Obsidian vault structure and formatting |
| templates/signal-note.md | Template for Signal vault notes |
| templates/reference-note.md | Template for Reference vault notes |
| templates/learn-note.md | Template for Learn vault notes |
| eval/checklist.md | Pass/fail tests before completing |
| eval/advisory-board.md | AI reviewer personas for quality assurance |
