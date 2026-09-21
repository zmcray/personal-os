# Phase 1: Orient

**Purpose:** Load the current state of memory so you know what you already know before gathering new signal.

---

## Steps

1. **Read CLAUDE.md** at the Work folder root. This is the index. Note the current structure, what it points to, and any sections that reference dates.

2. **Read all 00_Context files:**
   - `00_Context/about-me.md`
   - `00_Context/voice-and-style.md`
   - `00_Context/working-rules.md`

3. **Read active-memory.md** (`40_OS/08_Memory/active-memory.md`). If it doesn't exist yet, note that this is the first run and skip to gather.

4. **Scan 08_Memory directory** for all files. Read key reference files:
   - `glossary.md`
   - `people-quickref.md`
   - `scheduled-tasks.md`
   - `workblock-workstreams.md`

5. **Staleness check.** Flag any entries in active-memory.md where:
   - A date reference is older than 14 days with no refresh
   - A project or person is referenced that no longer appears in recent activity
   - A "pattern" or "observation" hasn't been validated or referenced in 2+ weeks
   - Any entry contradicts current CLAUDE.md or 00_Context content

6. **Log orient results internally** (not written to file, just held in context):
   - Number of memory files loaded
   - Stale entries flagged (with line references)
   - Any gaps noticed (e.g., a person mentioned in recent sessions but not in people directory)

---

## Orient should NOT:
- Modify any files
- Write output to chat
- Make decisions about what to keep or prune (that's Phase 4)
