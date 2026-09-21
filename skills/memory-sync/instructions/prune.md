# Phase 4: Prune

**Purpose:** Keep active-memory.md within its budget (200 lines, ~25kb). Archive overflow before removing. Force compression so only what actually matters stays.

---

## Steps

1. **Measure active-memory.md** after consolidation. Count lines and estimate file size.

2. **If within budget (under 200 lines and ~25kb):** Skip to step 5. No pruning needed.

3. **If over budget, select entries for archival.** Use the priority stack below to decide what moves to archive:

   **Keep (highest priority, last to be pruned):**
   - Corrections from the last 7 days
   - Active project status
   - People updates from the last 7 days
   - Open stale flags
   - Current week's context

   **Compress (reduce detail, keep the core fact):**
   - Corrections older than 7 days but younger than 14 days
   - Patterns observed (compress to one line each)
   - System changes older than 7 days

   **Archive (move to archive file):**
   - Corrections older than 14 days (they should be internalized by now or added to CLAUDE.md)
   - People updates older than 14 days (should be in people profiles by now)
   - Project status entries for completed or paused projects
   - Patterns that were never confirmed (observed once, never again)
   - System changes older than 14 days

4. **Write archived entries** to `40_OS/08_Memory/memory-archive-YYYY-MM.md` (using current month). Append, don't overwrite. Each archived batch gets a header:

   ```
   ## Archived 2026-04-02

   [entries, preserving their original dates and classification]
   ```

5. **Final check.** Re-count lines. If still over budget, compress more aggressively:
   - Merge related entries into single lines
   - Drop detail from older entries, keeping only the core fact
   - As last resort, archive entries from the "compress" tier

6. **Write the final pruned active-memory.md.**

---

## Archival Rules

- **Always append to the current month's archive.** Don't create a new file per day.
- **Preserve original dates on archived entries.** The archive is a chronological record.
- **Archive files are never pruned automatically.** They grow until the monthly audit (Mercer) reviews them.
- **If an archived entry becomes relevant again** (e.g., a paused project restarts), move it back to active-memory.md during consolidation.

---

## Prune should NOT:
- Delete archive files
- Prune CLAUDE.md or any other file
- Drop entries without archiving them first
- Remove entries that have been in active-memory.md for less than 3 days (give them time to prove relevance)
