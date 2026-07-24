# Loops Map

Source of truth for the closed-loop architecture of Zack's OS. Each loop mapped at mid-level depth. Built collaboratively across sessions; will integrate with or replace sections of `os-config.md` once stable. This file is the **index/map** across all loops; per-loop workflow detail lives in `40_OS/01_Workflows/loops/`.

**Last updated:** 2026-04-29 (annotation added 2026-07-05)
**Status:** Complete (13 of 13 loops mapped) ... full refresh pending (loop-triage item 15, held)

**Note (2026-07-05):** Loop mentions of the Notion CRM and Meeting Notes predate two migrations: CRM moved to Attio 2026-06-10, and the Meeting Notes DB was deprecated 2026-07-05. Current model: full debrief = Attio note on the person record (sole system of record) + Attio follow-up tasks; transcripts live in Granola, fetched via MCP on demand. Read this file as historical until the item-15 rewrite lands. A second migration also postdates this map: the Notion Roadmap moved to Supabase 2026-07-21 (see `supabase-roadmap-contract.md`); Roadmap mentions below describe the Notion era.

---

## Closed-Loop Framework

Every loop in the OS must have these 6 stages. Loops missing a stage are broken.

1. **TRIGGER** ... entry point (slash command, scheduled cron, inbound event, user phrase)
2. **CAPTURE** ... raw input lands somewhere ephemeral or staged
3. **PROCESS** ... a skill or workflow classifies, structures, transforms
4. **STORE** ... persisted to a system of record (Notion DB, file, Linear, Gmail)
5. **REVIEW** ... a downstream skill or person reads it back at the right time
6. **CLOSE** ... source state updated. Loop ends or feeds the next loop

Each loop entry follows a locked template:

```
[1. TRIGGER]
  How it starts:
  • [verb-led list]

[2. CAPTURE]
  Where the raw input lands:
  • [system defined inline on first mention]

[3. PROCESS]
  What gets done to it:
  • [verb-led actions, no skill name redundancy if already named in trigger]

[4. STORE]
  Where the outputs land:
  • [system → role description, no DB IDs]

[5. REVIEW]
  What reads it back later:
  • [skill (cadence) → what it does]

[6. CLOSE]
  How we know the loop ended:
  • [explicit state changes]

[BROKEN]
  • [actual gaps from skill files / observable system state]
```

---

## Index

1. Capture → Roadmap (universal capture)
2. Relationship / CRM
3. Email
4. Daily Rhythm
5. Memory
6. Signal / Knowledge Intake
7. Personal Vault Compile (raw → wiki)
8. McRay Group Vault (thinking → decisions → artifacts)
9. Content
10. Build Pipeline
11. Strategic Thinking
12. System Maintenance
13. Job Search

---

## 1. Capture → Roadmap

Universal capture loop. Encompasses 8 capture skills: /roadmap, /task, /buildnote, /thesis, /strategy, /brand, /learn, /flag-that.

```
[1. TRIGGER]
  How it starts:
  • Slash commands: /roadmap, /task, /buildnote, /thesis, /strategy, /brand,
    /learn, /flag-that
  • Natural language: "remind me to", "build note", "log a thought",
    "remember this pattern"

[2. CAPTURE]
  Where the raw input lands:
  • Desktop: appended directly to local files (vault thinking/ for thesis/strategy/
    brand, vault raw/ for learn-queue, feedback log for flag-that)
  • Mobile/web fallback: Notion Capture Inbox tagged by Type, status = Pending
  • Tasks/ideas: Notion Roadmap (3-layer: Workblock > Workstream > Task)
  • Builds: Linear Mcraygroup team (or Notion Cowork OS Backlog for OS-flavored)
  • Best practices: Notion Best Practices DB

[3. PROCESS]
  What gets done to it:
  • Auto-classify Type, Workstream, Workblock, source skill
  • Smart-route Status and Scheduled For (today for /task, Backlog for /roadmap)
  • Append with ET timestamp to vault files; never rewrite the input
  • Mobile captures: stage in Capture Inbox as Pending, no rewrite

[4. STORE]
  Where the outputs land:
  • Notion Roadmap → daily task tracking
  • Notion Best Practices → pattern log
  • Linear Mcraygroup → build pipeline (5-stage)
  • Notion Cowork OS Backlog → OS-flavored builds
  • McRay Group thinking/ → thesis-notes.md, strategy-notes.md, brand-notes.md
  • Vault learn-queue.md → learning questions
  • Feedback log.md → corrections and gotchas
  • Capture Inbox → mobile staging

[5. REVIEW]
  What reads it back later:
  • /sync-captures (in begin-the-day Layer 0.25) → drains Capture Inbox to vault
  • begin-the-day → today's Roadmap In Progress + Not Started
  • weekly-review (Friday) → Backlog by workstream, carry-forwards, stuck items
  • end-of-day → closes In Progress, rolls unfinished forward
  • signal-session → reads thesis-notes, strategy-notes for synthesis
  • Friday feedback rollup → reads feedback log, proposes gotcha updates

[6. CLOSE]
  How we know the loop ended:
  • Roadmap task → Done with Completed On date
  • Linear issue → moved through 5 stages to Stable
  • Capture Inbox row → marked Routed (audit trail kept)
  • Best Practice → captured (these are reference, not action)
  • Feedback log entry → rolled into a skill gotcha update or skill refinement

[BROKEN]
  • Mobile captures only flow if /sync-captures runs; if skipped, they sit Pending
  • Race condition: if mobile capture lands at 6 AM, Layer 0.25 may not finish
    before Layer 1 reads vault files
  • /study skill referenced in /learn, but appears to be future-state
  • Playbook captures route to Capture Inbox but vault destination path unclear
  • Buildnote project picker caps at 4 options ... projects beyond fall to "Other"
  • Best Practice DB has no read-back loop ... entries captured, never re-surfaced
```

---

## 2. Relationship / CRM

Call → debrief → store → review → next-touch → close.

```
[1. TRIGGER]
  How it starts:
  • You say "debrief" or name someone post-call
  • Granola (your call recorder) drops a fresh row into the system
  • End-of-day finds an unprocessed call from earlier

[2. CAPTURE]
  Where the raw input lands:
  • Granola path: Granola → n8n webhook → row in Meeting Notes
    (Notion DB, full call transcripts and metadata, the permanent record)
  • Voice/text path: pasted directly in chat

[3. PROCESS]
  What gets done to it:
  • Pull the contact's CRM dossier and prior call history
  • Structure the raw input into takeaways + action items
  • Draft a follow-up email
  • Extract anything actionable into discrete tasks

[4. STORE]
  Where the outputs land:
  • Meeting Notes (Notion) → full structured debrief, system of record
  • CRM (Notion) → lean summary + backlink, Last-Contact bumped
  • Gmail → follow-up draft saved for your review (not sent)
  • Roadmap (Notion) → action items written as tasks

[5. REVIEW]
  What reads it back later:
  • crm-auto-check (every morning) → flags stale contacts
  • daily-call-prep (morning) → .docx brief from CRM + Meeting Notes
  • thankyou (chains from debrief) → another Gmail draft
  • crm-lookup (on demand) → dossier when you ask "pull up [name]"

[6. CLOSE]
  How we know the loop ended:
  • CRM Last-Contact = today
  • Follow-up sent (Gmail) or scheduled (Roadmap)
  • Meeting Notes marked Processed

[BROKEN]
  • Granola rows that never get debriefed sit at Processed=false forever ... no GC
  • Gmail drafts get created but no one tracks whether they're sent
  • Last-Contact update is implicit ... needs verification it actually fires
```

---

## 3. Email

Inbound Gmail → triage → respond/archive → CRM update if relevant.

```
[1. TRIGGER]
  How it starts:
  • User phrase: "/email-triage", "/email [recipient]", "/reply-all"
  • Natural language: "triage email", "email [name] about", "any important emails"
  • Scheduled: crm-auto-check at 5:45 AM ET (silent, output read by begin-the-day)

[2. CAPTURE]
  Where the raw input lands:
  • Gmail Inbox: last 24 hours pulled via search
  • Email rules read from 40_OS/08_Memory/email-rules.md (VIPs, auto-archive, overrides)
  • Email triage history at 40_OS/08_Memory/email-triage-log.md (dedup protection)

[3. PROCESS]
  What gets done to it:
  • Cross-reference each sender against Attio CRM
  • Classify into 4 buckets: Reply Now, Review, Brief, Archive
  • CRM contacts floor at Review minimum (never auto-archive a known contact)
  • Apply rule overrides; flag unknown human senders as potential CRM adds
  • Wait for approval before any email moves

[4. STORE]
  Where the outputs land:
  • Gmail labels: Cora/Brief, Cora/Archived (auto-archived from inbox)
  • Reply Now and Review emails: stay in inbox untouched
  • Email-triage-log.md: full session audit trail
  • Attio CRM: new contact record if user approves an unknown-sender flag
  • Gmail Drafts: replies created by /email-draft or /reply-all

[5. REVIEW]
  What reads it back later:
  • /email-draft (on demand) → CRM context + voice rules → Gmail draft
  • /reply-all → batch HTML drafts with proper thread quoting
  • begin-the-day Layer 0.5 → surfaces crm-auto-check output
  • end-of-day Part 2 → checks overdue follow-ups, missing call prep

[6. CLOSE]
  How we know the loop ended:
  • Reply Now/Review: user manually sends or acts (external to system)
  • Brief/Archive: labeled and out of inbox
  • Unknown sender → CRM record created, recognized in next triage

[BROKEN]
  • /reply-all creates drafts but no audit that they get sent ... silent backlog risk
  • Email-triage-log.md is append-only, no GC ... grows unbounded
  • crm-auto-check is a separate scheduled task, not documented in email-triage skill
  • No live feedback if CRM lookup misses or fuzzy-matches wrong (defaults to Review)
```

---

## 4. Daily Rhythm

Three synchronized entry points (6 AM begin, 4:45 PM end, Friday 12:30 review) that bookend each day and week. Self-feeding: EOD writes tomorrow-focus, AM reads it; Friday writes next-week-plan, AM reads it Mon-Fri.

```
[1. TRIGGER]
  How it starts:
  • Scheduled: begin-the-day at 6 AM ET (auto, daily)
  • Scheduled: end-of-day at 4:45 PM ET (auto, daily)
  • Scheduled: weekly-review Friday 12:30 PM ET (auto, weekly)
  • Manual: "start my day", "wrap up", "weekly review"

[2. CAPTURE]
  Where the raw input lands:
  • Google Calendar (<work-email>) → today's events, calls, deep work blocks
  • Notion Roadmap → today's tasks (In Progress, Not Started)
  • Attio CRM → contact context for today's calls
  • Notion Capture Inbox → overnight mobile captures (drained Layer 0.25)
  • Tomorrow-focus.md → yesterday's EOD flag for today's deep work
  • Next-week-plan.md → Friday's block assignments for the week
  • Active-projects.md → live project snapshot
  • Signal Library → high-signal items connected to today
  • Meeting Notes → yesterday's Granola transcripts (EOD scans)

[3. PROCESS]
  What gets done to it:
  • Morning: drain Capture Inbox, promote Not Started to In Progress, surface
    today's calls with prep, recommend deep work block, surface signal feed
  • EOD: process unprocessed Granola rows, present In Progress as checklist
    (done/rollover), ask for tomorrow's deep work flag
  • Friday review: pull week's velocity, surface Backlog by workstream, lead
    priority conversation, assign 5 morning blocks for next week, run call prep
    for next week's calls, run feedback rollup

[4. STORE]
  Where the outputs land:
  • Tomorrow-focus.md → tomorrow's deep work flag (written by EOD)
  • Next-week-plan.md → 5 days of block assignments (written by Friday review)
  • Notion Roadmap → status updates (Done, Rollover Count++)
  • Google Calendar → event color/description edits, prep doc links
  • Attio CRM → contact pages updated with prep content, last-touch
  • Word docs → call prep briefs in 50_Network/01_Call Prep/

[5. REVIEW]
  What reads it back later:
  • Loop is self-feeding: EOD writes tomorrow-focus, AM reads it
  • Friday writes next-week-plan, AM reads it Mon-Fri
  • Feedback log entries flow into Friday rollup → gotcha updates

[6. CLOSE]
  How we know the loop ended:
  • Morning briefing delivered, Zack moves into deep work
  • EOD: all In Progress tasks disposed (Done or Rollover), tomorrow-focus written
  • Friday: next-week-plan written with 5 morning blocks, all calls have prep docs

[BROKEN]
  • Layer 0.25 (sync-captures) has no wait/block before Layer 1 vault reads
  • Call prep logic exists in 3 places: begin-the-day, weekly-review, daily-call-prep
    skill ... unclear authority
  • Tasks with Rollover > 2 silently return to Backlog ... no alert in briefing
  • Backlog priority conversation is skippable in Friday review ... items can rot
  • Protected deep work window (8:45-10:45 AM) is flagged on conflict but not
    actively enforced
  • Signal Library items not marked "briefed" → potential re-surface loop
  • EOD Part 2 surfaces overdue follow-ups but no automated action
```

---

## 5. Memory

Daily learning loop. Daily 6 PM consolidation pass + 2 AM overnight pass. Manual trigger via /memory-sync.

```
[1. TRIGGER]
  How it starts:
  • Scheduled: 6 PM ET daily (consolidation pass)
  • Scheduled: 2 AM ET overnight (decay/dedupe/pattern detect)
  • Manual: "/memory-sync", "sync memory", "consolidate memory"

[2. CAPTURE]
  Where the raw input lands:
  • Memory-sync gathers from: Notion activity (Roadmap, CRM, Meeting Notes),
    file changes in vault, calendar events, Granola transcripts, feedback log
  • All inputs land in memory-sync's working scope (not stored separately)

[3. PROCESS]
  What gets done to it:
  • Daily pass: orient (flag stale context) → gather signal → consolidate
    into active-memory.md with [w:N] weight + [src:] provenance tags →
    prune to 200-line budget, archive overflow to monthly archive file
  • Overnight pass: pipeline health check (Granola → Notion webhook integrity) →
    decay weights → dedupe → pattern detect → rehearsal promotion →
    write Patterns Observed block to overnight-log.md

[4. STORE]
  Where the outputs land:
  • active-memory.md → living 200-line context, weighted + tagged
  • memory-archive-YYYY-MM.md → monthly overflow archive
  • overnight-log.md → patterns observed during 2 AM pass
  • sync-log.md → record of each sync run (timing, deltas)

[5. REVIEW]
  What reads it back later:
  • Every Claude session reads active-memory.md at session start
  • begin-the-day reads overnight-log.md → surfaces patterns in briefing
  • Mercer (monthly audit) cross-checks active-memory against live tools,
    flags staleness, provenance gaps, drift
  • memory-sync next pass cycles context back in, refreshes weights

[6. CLOSE]
  How we know the loop ended:
  • Overnight pass writes timestamped block to overnight-log.md
  • 200-line budget reached, overflow archived with dated marker
  • Mercer audit confirms drift contained or surfaces it explicitly

[BROKEN]
  • Pipeline health check catches Granola webhook failures but only surfaces
    them in memory ... no auto-retry, no escalation
  • Decay/dedupe runs on 7-day window; older entries below decay threshold
    can vanish without explicit promotion. No audit trail of culled entries
  • No explicit signal back to user when active-memory drifts from reality
    until monthly Mercer audit catches it
```

---

## 6. Signal / Knowledge Intake

External content captured for later synthesis. Two ingest paths: automated (n8n-fed Signal Library) and manual (signal-review on URL/transcript paste).

```
[1. TRIGGER]
  How it starts:
  • Manual: paste URL or transcript + "review this for signal", "pull signal",
    "signal review", "anything good in this"
  • Automated: X bookmarks, Snipd podcast clips, Web Clipper articles flow
    into Signal Library via n8n
  • Scheduled: signal-classify-sync at 7 AM ET daily

[2. CAPTURE]
  Where the raw input lands:
  • Manual path: signal-review fetches source (YouTube via Chrome MCP, articles
    via WebFetch, PDFs/transcripts local), saves raw markdown to vault raw/
  • Automated path: items land in Notion Signal Library, untyped

[3. PROCESS]
  What gets done to it:
  • signal-review: extract 5-7 top ideas, check thesis alignment, propose
    capture candidates (/thesis, /strategy, /buildnote, /task)
    with exact text, wait for approval before firing downstream
  • signal-classify-sync (daily 7 AM): classify Signal Library items into
    3-bucket tree (Signal/Reference/Learn), sync to vault raw/

[4. STORE]
  Where the outputs land:
  • Vault raw/ → immutable source captures with frontmatter tags
  • Notion Signal Library → tagged Signal Type after classify-sync runs
  • Per-approval downstream destinations: McRay Group thinking/ files (thesis,
    strategy, brand), Linear (buildnote), Notion Best Practices, Roadmap

[5. REVIEW]
  What reads it back later:
  • signal-session → reads raw/ + Signal Library + wiki for synthesis
  • vault-linker (daily 7:49 AM) → auto-links unlinked raw notes to wiki hubs
  • Begin-the-day briefing surfaces Signal feed items connected to today

[6. CLOSE]
  How we know the loop ended:
  • Raw capture saved to vault raw/ with frontmatter tags
  • Signal Library item gets Signal Type set
  • vault-linker fills the Connections section in raw note with prose +
    relative markdown links to wiki articles

[BROKEN]
  • signal-review requires per-candidate approval ... no batching, friction on
    multi-candidate sessions
  • If fetch fails (paywall, YouTube block), fallback requires manual paste.
    No automatic retry or escalation
  • signal-classify-sync runs once daily at 7 AM, so newly reviewed items
    won't appear in vault raw/ until next morning
  • No signal back to user when an item never gets connected to a wiki hub
    by vault-linker (orphaned source)
```

---

## 7. Personal Vault Compile (raw → wiki)

The compile-forward architecture. raw/ (immutable sources) → wiki/ (LLM-compiled knowledge base) → outputs/ (ephemeral session work). **Key finding: there is no dedicated compile skill.** signal-session is the de facto compile mechanism, supplemented by vault-linker for cross-references.

```
[1. TRIGGER]
  How it starts:
  • Manual: "/signal", "let's pull threads", "what patterns are emerging",
    "connect the dots"
  • Implicit: signal-session always runs a compile pass at session close
  • Scheduled: vault-linker daily 7:49 AM

[2. CAPTURE]
  Where the raw input lands:
  • signal-session reads from: vault wiki/_index.md + concepts/sectors/
    patterns/evidence/players articles → Signal Library → past conversations →
    Attio CRM + Meeting Notes → web search to verify
  • Web Clipper clips auto-detected in raw/ tagged [raw, uncompiled]

[3. PROCESS]
  What gets done to it:
  • Open-Ended mode: surface 2-3 synthesis threads from recent captures,
    lead with analysis take
  • Directed mode: pull relevant captures, engage as thinking partner
    (pressure-test, extend, connect, challenge)
  • At session close: scan raw/ for uncompiled clips, incorporate into wiki,
    remove [uncompiled] tag, add compiled: [date] frontmatter

[4. STORE]
  Where the outputs land:
  • wiki/ → new or updated articles (concepts, sectors, patterns, evidence,
    players). Each article: YAML frontmatter + outbound links + raw refs
  • wiki/_index.md → updated with new article entries
  • outputs/ → session synthesis note with [[wiki article]] links
    (ephemeral, 7-day promote-or-prune window)
  • Notion Signal Library → closing note titled "Signal Session ... [Date] ... [Theme]"

[5. REVIEW]
  What reads it back later:
  • vault-linker (daily 7:49 AM) → detects new wiki articles from prior day,
    backfills cross-references in older raw notes
  • Friday weekly review → checks outputs/ for promotion candidates
  • begin-the-day → reads latest outputs/ and _index.md updates
  • Every signal-session reads wiki/_index.md first

[6. CLOSE]
  How we know the loop ended:
  • Wiki article exists with YAML + outbound links + raw refs
  • _index.md updated
  • Synthesis saved to outputs/ with [[wiki]] references
  • Closing note posted to Signal Library
  • Chat summary: count of wiki articles created/updated, clips compiled

[BROKEN]
  • No dedicated /compile skill to audit raw/ vs wiki/ coverage on demand.
    Compile is implicit in signal-session ... if you don't /signal, you don't compile
  • Web Clipper detection assumes clips land in raw/ with [raw, uncompiled].
    No validation that Web Clipper is configured correctly. Misconfigured =
    orphan
  • Same-day cross-references rely on manual linkage; vault-linker only
    catches them next morning at 7:49 AM
  • outputs/ has 7-day window but no automated promotion or prune.
    Manual action required each Friday or items decay silently
```

---

## 8. McRay Group Vault (thinking → decisions → artifacts)

Institutional knowledge for the firm. Three-stage promotion: thinking/ (raw append-only capture) → decisions/ (committed) → artifacts/ (published canonical). Most automated path: decision triage in Friday weekly review. Largely manual elsewhere.

```
[1. TRIGGER]
  How it starts:
  • Capture (covered in Loop 1): /thesis, /strategy, /brand
    append to thinking/ files
  • Decision capture: manual journal entry to decision-notes.md
    (no /decision skill exists yet)
  • Promotion triggers: Friday weekly-review triages decision-notes;
    artifact promotion is manual (move Draft → artifacts/ on ship)

[2. CAPTURE]
  Where the raw input lands:
  • thinking/thesis-notes.md, strategy-notes.md, brand-notes.md, playbook-notes.md
    → running append-only captures, ET-timestamped blocks
  • thinking/decision-notes.md → rough decision capture (manual)
  • 60_Strategy/thesis/*-Draft.md → working drafts of long-form thesis pieces

[3. PROCESS]
  What gets done to it:
  • Friday weekly-review (Part 3): read decision-notes from the week,
    evaluate committed vs exploratory, promote committed to decisions/
  • Manual draft-to-artifact: when a thesis draft ships, move from
    60_Strategy/thesis/foo-Draft.md to 50_Vault/artifacts/foo-vYYYYMMDD.md
  • Mercer (monthly): scans decision-log for drift, contradictions

[4. STORE]
  Where the outputs land:
  • thinking/ → running capture files (immutable append, never rewrite)
  • decisions/YYYY-MM-DD-decision-slug.md → standalone decision doc with
    context, rationale, stakeholders, implications
  • decisions/decision-log.md → chronological index, 1-line summary per entry
  • artifacts/ → canonical published reference (vision.md, service-offering.md,
    operating-principles.md, brand-principles.md, shipped thesis pieces)

[5. REVIEW]
  What reads it back later:
  • Friday weekly-review → triages thinking/decision-notes
  • Mercer monthly audit → scans decision-log for recency and contradictions
  • Luce (writing skill) → reads thinking/ + identity/ when drafting
  • Caspian (PRD) → reads thinking/ for relevant strategy context
  • Hagen (consigliere) → may reference identity/ + decisions/ when reasoning

[6. CLOSE]
  How we know the loop ended:
  • Decision: promoted to decisions/YYYY-MM-DD-*.md, indexed in decision-log
  • Thesis draft: shipped, moved to artifacts/ with date-versioned filename
  • Artifact: canonical only (no drafts in artifacts/)

[BROKEN]
  • No /decision skill exists. Decisions only captured by manual journal entry
  • Friday-only triage means a decision Tue-Thu sits in decision-notes for days
  • decision-log.md updates are manual ... a decision can be promoted to
    decisions/ but never indexed, making it invisible to readers
  • Decisions don't auto-update the thesis-notes or strategy-notes that
    justified them ... no bidirectional sync
  • Thesis draft → artifact promotion is fully manual; no skill enforces
    versioning, cross-link updates, or removal of stale Drafts
  • SOPs folder is a stub ... nothing written there yet, no skill targets it
```

---

## 9. Content

Idea → draft → publish → measure → refine. X primary, LinkedIn secondary.

```
[1. TRIGGER]
  How it starts:
  • Slash commands: /content, /reply, /performance
  • Natural language: "let's draft", "draft a post", "turn this into a post",
    "log engagement", "how did that post do"
  • Source material can be a signal-session output, thesis note, debrief, raw thought

[2. CAPTURE]
  Where the raw input lands:
  • Conversation (raw text, pasted idea, reference to prior output)
  • Notion Signal Library entries can feed as source material

[3. PROCESS]
  What gets done to it:
  • Identify entry mode (Raw Thought, Signal, Debrief, Thesis, Batch)
  • Map to content pillar (AI + Operations, PE Value Creation, Operator Lessons,
    Builder's Journey)
  • Pull evidence from vault wiki to ground the draft
  • Search Content Calendar to avoid repetition
  • Select format (tweet, X Article, LinkedIn post)
  • Draft 2-3 variations, run eval checklist + advisory board pressure-test
  • Present drafts for manual review before save

[4. STORE]
  Where the outputs land:
  • Notion Content Calendar (Status: Drafted or Ready, per pillar and platform)
  • Engagement metrics logged post-publish via /performance

[5. REVIEW]
  What reads it back later:
  • content-performance (after publish, logs metrics; reviews patterns at 15+ posts)
  • content-reply (scans for engagement opportunities)
  • Friday weekly-review surfaces published content performance
  • Mercer monthly audit checks content loop health

[6. CLOSE]
  How we know the loop ended:
  • Status moves to Ready, then Published with URL populated
  • Engagement metrics logged
  • Performance insights file updated with patterns for next draft cycle

[BROKEN]
  • Performance insights → content-engine feedback is manual (not auto-pulled)
  • Content Calendar dedup is manual Notion query, not automated alert
  • Readwise ingest discontinued 2026-04-08 ... Web Clipper articles land in
    vault raw/ instead. Governance gap on what counts as a content source
  • No automated check that drafted posts actually publish (Drafted can stall
    indefinitely)
```

---

## 10. Build Pipeline

Idea → Scoped → Building → Live (Beta) → Stable. Linear is system of record.

```
[1. TRIGGER]
  How it starts:
  • Slash commands: /buildnote (capture), /caspian or /prd (formal scoping)
  • Natural language: "build note", "let's PRD this", "should we build X"
  • Project hint via prefix (buildnote - projectname), inline mention, or context

[2. CAPTURE]
  Where the raw input lands:
  • Conversation text after /buildnote or /caspian
  • Linear (Mcraygroup team) is the system of record since 2026-04-28
  • OS-flavored buildnotes route to Notion Cowork OS Backlog instead

[3. PROCESS]
  What gets done to it:
  • Buildnote: extract note as-is, resolve project (prefix > inline > context),
    fetch live Linear projects to confirm, ask if ambiguous
  • Caspian (when invoked): 8-phase flow ... Frame (PG validation), Imagine
    (Bezos PR), Calibrate (Tan posture), Build (Cagan four risks), Cut (Jobs
    focus), Render (PRD draft), Approve & Ship (write disk + Linear + Registry)
  • Issue created in Backlog state, no priority/labels assigned at capture

[4. STORE]
  Where the outputs land:
  • Linear Issue (Mcraygroup, Backlog, project-tagged)
  • Notion Cowork OS Backlog if OS-routed
  • Caspian PRDs: 01_McRayGroup/30_Projects/<project>/docs/strategy/
    YYYY-MM-DD-<topic>-PRD.md
  • Caspian firm-level: 01_McRayGroup/60_Strategy/<theme>/YYYY-MM-DD-<topic>.md
  • Linear Initiative + Issues per MVP feature
  • Notion Project Registry updated (Latest PRD, Initiative ID, Last Refreshed)
  • Caspian session file: 40_OS/08_Memory/caspian-sessions/active/

[5. REVIEW]
  What reads it back later:
  • Strategy sessions (Tue/Thu) triage Backlog → Todo/In Progress
  • Friday weekly-review pulls Linear into next week
  • Pulse Build tab (live kanban from Linear, 60s cache) is the dashboard
  • Caspian REFRESH/EXPAND modes run drift detection on prior PRD
  • Mercer monthly audit checks Backlog and PRD health

[6. CLOSE]
  How we know the loop ended:
  • Buildnote: Issue moves Backlog → Todo → In Progress → Done
  • Caspian PRD: file written, Linear IDs back-written to frontmatter,
    session moved from active/ to completed/
  • Build progresses through 5 stages (encoded in Linear project content)
    to Stable

[BROKEN]
  • Cowork OS removed from Linear by design but no clear merge strategy if
    OS work needs to interact with Linear-tracked work
  • Buildnote project picker caps at 4 options ... projects beyond fall to "Other"
  • Caspian Killed features stored only in PRD Decision Log graveyard,
    no persistent archive for cross-PRD pattern analysis
  • No auto-retry on Phase 8 partial failure (manual resume required)
  • No scheduled sync from orphan buildnote Issues back to PRD for refresh
```

---

## 11. Strategic Thinking

Pressure-test, structure, write. Four personas: Hagen (consigliere), Caspian (PRD), Luce (writing), Minto (pyramid pressure-test).

```
[1. TRIGGER]
  How it starts:
  • Slash commands: /hagen, /caspian, /luce, /minto
  • Natural language: "pressure-test this", "let's think through this",
    "build this argument", "structure this", "is this MECE"
  • Defense mode (Luce): "defend this argument" or "attack this"
  • Caspian also covered in Loop 10; included here for the thinking dimension

[2. CAPTURE]
  Where the raw input lands:
  • Conversation (topic, decision, draft, argument)
  • Vault wiki articles oriented at session start
  • Prior drafts (Luce reads from vault outputs/luce/, McRay Group thesis drafts)
  • Prior PRDs (Caspian REFRESH/EXPAND)

[3. PROCESS]
  What gets done to it:
  • Hagen: six-archetype pressure-test (counselor, skeptic, advisor,
    pattern-spotter, principle-defender, contrarian); take first, support second
  • Caspian: 8-phase PRD flow blending five voices (PG, Bezos, Tan, Cagan, Jobs)
  • Luce: thesis-driven writing in 3 modes (Discovery, Build, Defense);
    structured arguments grounded in evidence
  • Minto: pyramid extraction (one-sentence answer, 2-4 MECE arguments,
    evidence per arm); diagnose gaps; deliver visual HTML artifact

[4. STORE]
  Where the outputs land:
  • Hagen: vault outputs/ session synthesis with [[wiki links]],
    wiki article updates if new thesis emerges
  • Caspian: PRD on disk + Linear Initiative + Issues + Project Registry
    (covered in Loop 10)
  • Luce: vault outputs/luce/ for working drafts; shipped pieces promote to
    01_McRayGroup/50_Vault/artifacts/ with date version
  • Minto: in-chat HTML artifact only (no persistent file)
  • Wiki updates from any of the four when thesis develops durably

[5. REVIEW]
  What reads it back later:
  • memory-sync (nightly) pulls session insights into active-memory.md
  • Caspian Phase 1 may reference prior Hagen outputs
  • Luce Defense mode reads prior Build drafts to attack
  • Friday weekly-review surfaces stalled drafts in 60_Strategy/thesis/
  • Mercer monthly audit checks decision/thesis trail integrity

[6. CLOSE]
  How we know the loop ended:
  • Hagen: synthesis saved with wiki links, one meta-observation delivered
  • Caspian: PRD shipped + Linear created (covered in Loop 10)
  • Luce: draft moved from -Draft to versioned artifact in artifacts/
  • Minto: visual artifact rendered, restructuring plan delivered

[BROKEN]
  • Wiki compile from Hagen relies on manual article creation
  • Hagen sessions don't auto-feed caspian (no event trigger if a hagen
    session reveals a PRD opportunity)
  • Premise challenges from Hagen are local; no central Decision Log links
  • Luce drafts can sit in 60_Strategy/thesis/ as -Draft indefinitely;
    no promotion enforcement
  • Minto produces no persistent record ... pyramid analysis evaporates
    after the artifact closes
```

---

## 12. System Maintenance

Keep the OS healthy. Monthly audit, ad-hoc cleanup, skill development, system documentation.

```
[1. TRIGGER]
  How it starts:
  • Slash commands: /audit (monthly), /folder-cleanup, /skill-creator,
    /doc-that, /memory-sync (covered in Loop 5)
  • Scheduled: Mercer auto-fires 1st of month
  • Natural language: "system check", "anything drifting", "clean up files",
    "create a skill", "document this", "consolidate memory"

[2. CAPTURE]
  Where the raw input lands:
  • Mercer: file system scan (Work folder), memory files, all Notion DBs,
    Linear projects + issues, skills index, vault articles, prior audit history
  • Folder-cleanup: Work folder root + subfolders for loose files
  • Skill-creator: existing skills inventory + new skill brief from conversation
  • Doc-that: the just-built system or component being documented

[3. PROCESS]
  What gets done to it:
  • Mercer: 7 modules ... Files & Folders, Memory & Context Integrity, Skills
    & Automations, Closed-Loop & Optimization Analysis, System Health Summary,
    Trend Tracking, Wiki Health
  • Folder-cleanup: classify loose files, propose move/archive/delete (never
    delete without approval), execute on confirm
  • Skill-creator: scaffold new skill from template, run evals on existing
    skills, optimize skill descriptions for trigger accuracy
  • Doc-that: extract system architecture from session, generate Word doc,
    update Build Log entry

[4. STORE]
  Where the outputs land:
  • Audit reports: timestamped markdown in Work folder
  • audit-history.md: monthly metrics (file count, skill count, open loops,
    auto-fixes, wiki growth, orphan count)
  • Files moved/archived per folder-cleanup approvals
  • Skills: 40_OS/05_Skills/<skill>/ folders with SKILL.md, instructions/,
    .skill packaged copy in dot_skills/
  • Word docs from doc-that in 40_OS/01_Workflows/01_Component Docs/
  • Notion Build Log database (one row per documented component)
  • Skill CHANGELOG entries

[5. REVIEW]
  What reads it back later:
  • Next month's Mercer compares to prior via audit-history.md
  • Friday weekly-review references audit findings
  • Caspian drift detection cross-checks audit findings
  • Skill-creator recommendations may trigger new skills in next sprint

[6. CLOSE]
  How we know the loop ended:
  • Audit report filed, audit-history.md updated, top 3 recommendations surfaced
  • Folder-cleanup: files relocated or archived, root clean
  • Skill-creator: skill packaged to .skill, CHANGELOG updated, present_files run
  • Doc-that: Word doc shipped + Build Log entry created

[BROKEN]
  • Mercer Module 4 produces optimization ideas with no tracking of pickup/reject
  • Mercer Module 5 recommendations not linked to skill-creator queue
  • Wiki health findings (orphans, contradictions) documented but no
    automated remediation
  • Trend tracking metrics captured but no visualization or out-of-pattern alert
  • No policy codified for folder-cleanup edge cases (move vs archive vs link)
  • Memory provenance audit flags missing [src:] tags but no bulk-repair process
  • Skill-creator evals exist but no scheduled regression testing across skills
  • Build Log database not surfaced in any workflow except doc-that itself
```

---

## 13. Job Search

Omitted from this public copy (personal). Mapped to the same 6-stage template as the loops above.

---

## Cross-Loop Patterns Observed

After all 13 loops, the recurring failure modes:

1. **Captured-never-read.** Best Practices, Roadmap Backlog items, Signal Library items, decision-notes (until Friday) can land and never re-surface in a timely way. No automated review cadence forces them back into view.

2. **Drafted-never-sent.** Gmail drafts (from email-draft, reply-all, thankyou) get created but no audit confirms they leave the drafts folder.

3. **Implicit state updates.** CRM Last-Contact bump, Meeting Notes.Processed flip, decision-log index entries happen (or don't) inside skills but are not verified. Silent failure surface.

4. **Skill authority overlap.** Call prep logic exists in 3 places (begin-the-day, weekly-review, daily-call-prep). signal-session doubles as the de facto vault compile mechanism. When multiple skills could fire, the OS doesn't designate which is canonical.

5. **No GC on append-only logs.** Email-triage-log, feedback log, vault thinking/ files, sync-log, overnight-log all grow unbounded. No archive cadence.

6. **Race conditions in begin-the-day.** Layer 0.25 (sync-captures) has no wait gate before Layer 1 vault reads. Mobile captures landing at 6 AM may miss the morning briefing.

7. **Missing skills hold loops open.** /study referenced in /learn but not built. /decision referenced in McRay Group vault flow but not built. Both leave their loops without instant capture.

8. **No dedicated compile mechanism.** raw/ → wiki/ relies on signal-session running. If you don't /signal, the vault doesn't compile. vault-linker fixes cross-refs but doesn't synthesize new wiki articles.

9. **Cross-loop event handoffs are implicit.** signal-classify-sync finishes at 7 AM, vault-linker runs at 7:49 AM, signal-session is on-demand. No event registry tracks which loop output should fire which downstream loop. Orphans accumulate.

10. **Stub folders.** SOPs folder in McRay Group vault has no content and no skill targets it. Same risk for any folder created without a writer skill.

11. **Database fragmentation.** Content lives in Notion Content Calendar, builds live in Linear with Notion fallback (OS Backlog), PRDs live on disk, system docs in Notion Build Log. No unified datastore. Cross-system queries require manual correlation.

12. **Wiki and memory are passive.** Hagen sessions document to wiki but Caspian doesn't pull from recent Hagen outputs. Mercer produces wiki findings but no skill triggers on wiki gaps. Vault is reference-rich but not active in decision workflows.

13. **Persona output asymmetry.** Caspian writes to disk + Linear + Notion. Hagen writes to vault outputs/ + wiki. Luce writes to vault outputs/luce/ + artifacts/. Minto writes nothing persistent. Output destinations don't align, so cross-skill review is hard.

14. **Frozen loop with no archive.** One loop is FROZEN but its DB and skill remain hot. No clear policy on what to do with paused loops (preserve, archive, soft-delete state).

15. **No closed-loop verification on the system itself.** Mercer audits the system but no skill audits Mercer. The audit history file grows but nothing reads it for compounding insight.

---

## Top 5 Refactor Priorities (preliminary)

Based on the cross-loop patterns. Rank-ordered by leverage (impact × frequency).

1. **Build the missing skills.** /study and /decision are referenced by other loops but don't exist. Both are instant-capture skills, low build cost, high downstream unblock value.

2. **Add a /compile skill.** Make raw → wiki an explicit, on-demand operation. Today it depends on /signal running. Adding /compile decouples the compile mechanism from the synthesis session.

3. **Sent-or-not audit on Gmail drafts.** Email-draft, reply-all, and thankyou all create drafts that may never leave Drafts. Add a daily check that flags drafts > 24 hours old.

4. **Cross-loop event registry.** Track which loop outputs feed which downstream loops. Today, signal-classify-sync (7 AM) → vault-linker (7:49 AM) → signal-session (on demand) is implicit. A registry makes orphans visible.

5. **GC policy for append-only files.** Email-triage-log, feedback log, vault thinking files, sync-log, overnight-log all grow unbounded. Add monthly archive (mirror of memory-archive-YYYY-MM.md pattern).

These are starting points. Final priorities should be re-derived once the artifact view exposes the structure visually.
