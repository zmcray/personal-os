---
name: vault-linker
description: Auto-link new vault notes to wiki hubs, peer signals, and people using relative markdown paths
schedule: 7:49 AM ET, daily
---

# Vault Linker

Auto-link unlinked notes in the Obsidian vault to wiki hubs, peer signals, and related people. Runs at 7:49 AM ET daily, after signal-classify-sync has pulled new X bookmarks and Snipd clips from the Notion Signal Library and written classified notes to `raw/`. Also picks up anything that landed directly in `raw/` (Obsidian Web Clipper articles, manual drops, `/thesis` and `/strategy` captures).

---

## Vault Architecture

The vault lives at `40_OS/09_Vault/` and uses a compile-forward structure:

```
09_Vault/
  raw/          <- All source notes (signals, references, thesis captures, strategy notes, clips)
  wiki/         <- Compiled knowledge base (the "hubs")
    _index.md   <- Master index, read first every session
    concepts/   <- Thesis pillars and frameworks
    sectors/    <- Vertical/horizontal market maps
    people/     <- Key people and thinkers
    companies/  <- Firms, portcos, startups, public companies
    patterns/   <- Recurring dynamics and models
    evidence/   <- Proof points and case studies
  outputs/      <- Ephemeral session work (luce drafts, etc.)
  research/     <- Deep-dive research folders
```

**Raw notes** are immutable source material. They have frontmatter tags and a `## Connections` section that vault-linker fills.

**Wiki articles** are the compiled hubs. They have frontmatter tags and serve as anchor points in the knowledge graph. Every link from a raw note to a wiki article strengthens that connection.

---

## Link Format (Canonical)

All links in the vault use **relative markdown paths**, not Obsidian wikilinks.

From `raw/` to wiki hubs:
```markdown
- [Agent Harness Engineering](../wiki/concepts/Agent Harness Engineering.md)
- [Specialty Distribution](../wiki/sectors/Specialty Distribution.md)
- [Todd Saunders](../wiki/people/Todd Saunders.md)
- [Constellation Software](../wiki/companies/Constellation Software.md)
- [Adoption Velocity](../wiki/patterns/Adoption-Velocity.md)
- [Operator-Led Adoption](../wiki/evidence/Operator-Led Adoption.md)
```

From `raw/` to peer raw notes (same directory):
```markdown
- [2026-03-23-todd-saunders-blue-collar-builders-movement](./2026-03-23-todd-saunders-blue-collar-builders-movement.md)
```

Never use `[[wikilinks]]`. Always use `[Display Name](relative/path/to/file.md)`.

---

## Workflow

### Step 0: Move Web Clipper Downloads to raw/

Before scanning the vault, check for Obsidian Web Clipper saves. These are `.md` files with `tags: clippings` or `tags:\n  - "clippings"` in their YAML frontmatter.

**Where to look:** Web Clipper saves to the user's `~/Downloads/` folder on the host machine. In the sandbox environment, this path is not directly accessible. Use the host file tools (Read/Glob) to scan `/Users/<user>/Downloads/` for `.md` files, then check frontmatter for the clippings tag.

For each match:
1. Read the frontmatter to extract the `title` field
2. Rename to `clip-` prefix + kebab-case title (e.g., `clip-how-we-run-25-person-company.md`)
3. Move from `~/Downloads/` to `40_OS/09_Vault/raw/`
4. Log what was moved

If no clipping files found, skip silently.

### Step 1: Read Gotchas

Read `gotchas.md` before doing anything else. These are known failure patterns from past runs.

### Step 2: Scan for Unlinked Notes

Read `instructions/note-scanning.md` for the rules.

Scan `raw/` for .md files where the `## Connections` section contains only a placeholder HTML comment (`<!-- To be filled by vault-linker -->`).

Output: List of unlinked notes with tags and metadata.

### Step 3: Build the Hub Index

Read `instructions/hub-indexing.md` for the rules.

Read all .md files in `wiki/concepts/`, `wiki/sectors/`, `wiki/people/`, `wiki/companies/`, `wiki/patterns/`, `wiki/evidence/`. Extract tags from frontmatter. Build a tag-to-hub lookup. Note: `wiki/companies/` articles use distinct frontmatter (type, sector, stage, key_people) and may have no tags; match them by company name and content.

Always re-read actual files. Do not rely on stale hub lists.

Output: Complete tag-to-hub index, logged.

### Step 4: Link Each Note

Read `references/linking-algorithm.md` for the algorithm overview.
Read `instructions/linking-rules.md` for the detailed linking logic.

For each unlinked note:
1. Extract tags from frontmatter
2. Find wiki hub notes with 2+ shared tags
3. Find peer raw notes with 2+ shared tags (cap at 4 peers)
4. Check if note references anyone from `wiki/people/` or any company from `wiki/companies/`
5. Fill `## Connections` with: 1-2 sentence prose connecting to Zack's thesis, then a bullet list of 2-8 relative-path links (hubs first, peers second, people and companies last)
6. Validate: exact filenames, relative paths, no wikilinks, no em dashes, only replace placeholders

Output: Updated notes saved to vault.

### Step 5: Detect Emerging Clusters

Read `instructions/cluster-detection.md` for the rules.

**Important:** Scan tag co-occurrence across ALL raw/ notes, not just the newly linked batch. Clusters build over time and a single run may only add 1-2 notes to an emerging pattern. Cross-reference against the hub index to find tag combinations with 3+ co-occurrences that no hub covers.

Output: List of emerging clusters with suggestions.

### Step 6: Generate Report

Read `instructions/reporting.md` for the format.

Create a brief report including:
- Count of notes linked
- Hub reference frequency (top 3)
- Emerging clusters flagged (with suggestions)
- Skipped notes (with reasons)

Keep report under 200 words, scannable format.

### Step 7: Evaluate Quality

Read `eval/checklist.md` and `eval/advisory-board.md`.

Run the checklist against all linked notes. Have the three personas (Librarian, Synthesizer, Cartographer) review the output before delivery.

---

## Files to Reference

**Instructions (read in order):**
- `instructions/note-scanning.md` ... how to identify unlinked notes
- `instructions/hub-indexing.md` ... how to build the hub index
- `instructions/linking-rules.md` ... the actual linking logic
- `instructions/cluster-detection.md` ... how to flag emerging clusters
- `instructions/reporting.md` ... report format

**References:**
- `references/linking-algorithm.md` ... algorithm overview and matching logic
- `gotchas.md` ... known failure patterns

**Templates:**
- `templates/linked-signal-note.md` ... example of a fully-linked signal note
- `templates/linked-reference-note.md` ... example of a fully-linked reference note

**Evaluation:**
- `eval/checklist.md` ... pass/fail tests before delivery
- `eval/advisory-board.md` ... quality review personas

---

## Success Criteria

Delivered output includes:
- All unlinked raw notes now have filled `## Connections` sections with prose and links
- All links use relative markdown paths (no wikilinks)
- No placeholder HTML comments remain in linked notes
- Hub links use `../wiki/subdir/File.md` format
- Peer links use `./filename.md` format
- Report is brief, scannable, and includes emerging cluster suggestions
- No links are broken (all target files exist)
- No existing content was overwritten
- No em dashes in any edited content
