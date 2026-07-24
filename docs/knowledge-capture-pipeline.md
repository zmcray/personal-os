# Knowledge Capture Pipeline Architecture

**Current state as of:** 2026-04-08
**Supersedes:** The April 6 "Readwise-Free" planning doc. Readwise has been removed from the pipeline. This document describes the live architecture.

---

## 1. Purpose

Defines the live architecture for Zack's knowledge capture pipeline: how raw signal enters the system, where it lands, how it gets classified, and how it becomes compiled knowledge in the Obsidian vault.

---

## 2. Design Principles

1. **Two destinations, one classifier.** Articles land in the Obsidian vault `raw/` directly via Web Clipper. X bookmarks and podcast clips land in the Notion Signal Library first. signal-classify-sync reads from Notion, classifies items, and writes classified Signal/Reference items into vault `raw/`. After that, vault-linker picks up everything new in `raw/` regardless of origin.
2. **No paid middle-layer.** Readwise was removed 2026-04-08 because it added a stall point without adding value. All ingest paths now run on native integrations or self-hosted automation (n8n webhook, Snipd's native Notion integration, Obsidian's Web Clipper extension).
3. **Event-driven where possible, polling only when necessary.** The X bookmark path fires on event. Snipd syncs asynchronously when you clip. Web Clipper is user-triggered per article.
4. **Idempotent downstream.** signal-classify-sync and vault-linker both dedupe by content identity, so re-running them is safe.

---

## 3. Ingest Paths

| Source | Capture Tool | Path | Destination |
|--------|--------------|------|-------------|
| X bookmarks | X API via n8n webhook | Bookmark fires webhook, n8n writes to Notion Signal Library with URL, author, content | Notion Signal Library |
| Web articles (desktop) | Obsidian Web Clipper | Full article as .md with images, auto-tagged `uncompiled` | Vault `raw/` direct |
| Web articles (iOS) | Obsidian Web Clipper (Safari) | Same as desktop. Open in Safari first if reading in-app. | Vault `raw/` direct |
| Podcast clips | Snipd native Notion integration | Clip in Snipd, Snipd writes direct to Notion | Notion Signal Library |
| Manual drops | Drag-and-drop into vault | PDFs, transcripts, screenshots | Vault `raw/` direct |
| Quick thoughts | `/thesis`, `/strategy` slash commands | Claude writes directly to vault | Vault `raw/` direct |

**Summary:** Two of six paths (X, Snipd) land in the Notion Signal Library. The other four (Web Clipper desktop, Web Clipper iOS, manual drops, slash commands) land directly in the vault `raw/`.

---

## 4. Flow Diagram

```
X Bookmarks ------> X API webhook -----> n8n -------------> Notion Signal Library
Snipd clips -------------------------------------> Notion Signal Library
                                                              |
                                                              v
                                             signal-classify-sync (daily ~7:30 AM ET)
                                                              |
                                                              v
Web Clipper (articles) -----> Vault raw/   <----- classified Signal/Reference items
Manual drops ---------------> Vault raw/
/thesis, /strategy ---------> Vault raw/
                                                              |
                                                              v
                                              vault-linker (daily ~7:49 AM ET)
                                                              |
                                                              v
                                              Linked to wiki hubs, concepts, people
                                                              |
                                                              v
                                        weekly-compile (Sun 7 PM ET) compiles raw/ into wiki/
```

---

## 5. Scheduled Jobs

| Schedule | Task | What it does |
|----------|------|--------------|
| Event-driven | X API webhook | Fires on bookmark save, n8n writes to Notion Signal Library |
| Asynchronous | Snipd → Notion | Native Snipd integration writes clips to Signal Library when you clip |
| User-triggered | Obsidian Web Clipper | Fires per article clip, writes to vault raw/ |
| Daily ~7:30 AM ET | signal-classify-sync | Pulls unclassified items from Notion Signal Library, classifies as Signal or Reference, writes classified items to vault raw/. (A legacy scheduler task ID from the pre-rename era is still live, pending rename.) |
| Daily ~7:49 AM ET | vault-linker | Links new vault raw/ notes to wiki hubs, concepts, and people |
| Sunday 7 PM ET | weekly-compile | Full compile pass across all uncompiled raw/ sources into wiki/. Updates _index.md, runs integrity check. |

---

## 6. Failure Modes and Monitoring

- **X API webhook stalls:** Monitor n8n workflow health. If X bookmarks stop appearing in the Signal Library for 24+ hours with no corresponding stop in Zack's bookmarking activity, check the webhook.
- **Snipd integration stalls:** Snipd has its own dashboard. Check there first.
- **Web Clipper failures:** User-visible. The clip either works or doesn't. No silent stall.
- **signal-classify-sync stalls:** schedule-watchdog (10:10 PM daily) flags missed runs.
- **vault-linker stalls:** Same watchdog.
- **weekly-compile skips:** Watchdog catches it the following Monday.

---

## 7. What Got Removed (2026-04-08)

- **Readwise product** ... subscription canceled, no longer in the ingest path
- **n8n Readwise-to-Notion sync workflow** ... disabled and archived (`40_OS/03_Automations/n8n/04-readwise-notion-sync.json` retained as reference)
- **readwise-classify-sync skill files** ... being consolidated with signal-classify-sync by Zack in a separate session

Historical Obsidian notes with `source: readwise` frontmatter are **preserved as provenance**. They are immutable captures of items that entered the vault via the old path. Do not rewrite them.

---

## 8. Related Documents

- Live memory state: `40_OS/08_Memory/active-memory.md`
- Vault architecture: `40_OS/08_Memory/obsidian-vault.md`
- Scheduled task reference: `40_OS/08_Memory/scheduled-tasks.md`
- Signal Library schema: `40_OS/05_Skills/shared/notion-databases.md` (Signal Library section)
- Signal Library DB ID: `collection://<NOTION_COLLECTION_ID>`

---

**Next update trigger:** If a new ingest path is added (e.g., podcast transcripts from a different source), add a row to Section 3 and update the flow diagram in Section 4.
