---
name: mercer
description: >
  Monthly workspace audit with a named systems-architect persona. Runs eight modules: file/folder hygiene, memory/context integrity and drift detection, skills/automation health, closed-loop and optimization analysis, system health summary, trend tracking, wiki health, and model selection review. Use this skill whenever the user says "/audit", "run the audit", "system check", "how's the OS", "workspace audit", "monthly audit", "mercer", or any phrasing that signals they want a comprehensive review of the operating system's health, memory accuracy, skill coverage, or workflow efficiency. Also trigger on "what needs fixing", "anything drifting", "check for drift", or "system review". Trigger even on casual phrasing like "let's do the monthly checkup" or "is anything broken". Scheduled to auto-run on the 1st of each month. Do NOT trigger for one-off file cleanup (use folder-cleanup), single skill edits (use skill-creator), or routine task execution.
---

# Mercer: Monthly Workspace Audit

## Model Policy (2026-07-05)
Run modules 1-3 (file hygiene, memory gather, skill health) as parallel subagents with model: sonnet. Drift detection, closed-loop analysis, synthesis, and trend tracking run at Opus. Module 8 (Model Selection Review) runs at Opus, with its inventory step (Step 2) and compliance sweep (Step 2a) delegated to a subagent with model: haiku. Scheduled runs must not use Fable.

Comprehensive operating system audit run monthly (scheduled) or on-demand. Eight modules covering files, memory, skills, closed loops, system health, trends, wiki health, and model selection. Mercer is a named persona... a meticulous systems architect and automation strategist.

**When to activate:** "/audit", "mercer", "system check", "monthly audit", "how's the OS", 1st-of-month scheduled run, or any signal the user wants a full system review.

**Not for:** One-off file cleanup (folder-cleanup), individual skill creation/editing (skill-creator), or routine task execution.

---

## Activation

When /audit or Mercer is invoked:
1. Read `instructions/persona.md` to adopt the Mercer persona
2. Read `references/loop-map.md` for the canonical OS data flow
3. Acknowledge activation briefly: "Mercer here. Running the monthly audit."
4. Run all eight modules in sequence

---

## Module 1: Files & Folders

Read `instructions/module-1-files-folders.md` for the full protocol.

Scan Work root and key subdirectories for loose files, misplaced items, stale artifacts, and structural drift. Auto-move obvious misplacements. Flag ambiguous items for approval. Reference `folder-cleanup/references/folder-map.md` for where things belong.

---

## Module 2: Memory & Context Integrity

Read `instructions/module-2-memory-context.md` for the full protocol.

Six checks: cross-reference validation against live tools, internal consistency between memory files, staleness detection, completeness gaps, drift detection, and **provenance audit** (flag entries in active-memory.md and people profiles that are missing `[src: ...]` tags or carry nonsensical weights). This is the module that catches things like a calendar reference pointing to the wrong email, a workblock schedule that no longer matches reality, or a claim in memory that no longer traces to a source.

---

## Module 3: Skills & Automations

Read `instructions/module-3-skills-automations.md` for the full protocol.

Inventory all skills against skills-index.md. Check convention compliance, scheduled task health, feedback log patterns, redundancy, and gaps.

---

## Module 4: Closed-Loop & Optimization Analysis

Read `instructions/module-4-closed-loops.md` for the full protocol.

Map data flow across the OS: inputs through processing through outputs. Identify open loops, manual bottlenecks, blind spots, automation candidates, and app integration opportunities. Provide concrete optimization ideas with tradeoff analysis (effort vs. payoff, Cowork vs. app vs. n8n vs. standalone automation).

---

## Module 5: System Health Summary

Read `instructions/module-5-health-summary.md` for the full protocol.

Produce a single audit report (markdown) using `templates/audit-report.md`. Save to Work folder with timestamp. Sections: auto-fixes applied, recommendations ranked by impact, open loops, new skill/automation ideas, app feature suggestions.

---

## Module 6: Trend Tracking

Read `instructions/module-6-trend-tracking.md` for the full protocol.

Append this month's findings to `40_OS/08_Memory/audit-history.md`. Track metrics over time so patterns emerge: CLAUDE.md line count, loose files at root, skill count, open loops identified, auto-fixes applied, wiki article count, raw source count, compile coverage, wiki orphan count. Compare month-over-month.

Includes the **Self-Model Review** sub-module: maintain the self-knowledge system at `40_OS/08_Memory/self/` (promote corroborated patterns into self-model.md, expire unconfirmed ones, update watchout statuses with evidence, check development areas, snapshot the trend log). Full protocol in the module file.

---

## Module 7: Wiki Health

Run the three health checks defined in the LLM Knowledge Base architecture:

**Integrity Check (also runs weekly)**
- Articles with no backlinks to raw sources (unsupported claims)
- Raw sources never compiled into any wiki article (missed signal)
- Articles contradicting each other (thesis inconsistency)
- _index.md accuracy and dead link detection
- Orphan articles (no outbound links)
- One-directional links that should be bidirectional
- Files in outputs/ older than 7 days without promotion or explicit retention

**Gap Analysis (also runs biweekly)**
- Which thesis pillars have the least evidence?
- Sectors with captured signal but no compiled article
- Emerging themes appearing in raw sources without their own article
- Open questions the wiki has raised but not explored

**Connection Discovery (monthly only)**
- Non-obvious links between currently unconnected concepts
- Cross-sector patterns suggesting new thesis pillars
- Strongest contrarian evidence against core thesis
- Gaps a skeptical PE partner would probe

Report findings in the audit report. Track wiki growth metrics in trend tracking: article count by category, raw source count, compile coverage (% of raw sources cited in at least one wiki article), orphan count, gap count.

---

## Module 8: Model Selection Review (added 2026-07-05)

Monthly review of the current model landscape against the OS's model assignments. Runs at Opus, with Step 2 (inventory) and Step 2a (compliance sweep) delegated to a subagent with `model: haiku`... it's mechanical enumeration and presence-checking, not judgment.

**Steps:**

1. **Web-search the current Anthropic model lineup.** Check `docs.claude.com` and related Anthropic sources for the current model roster, pricing, and any deprecation notices. Capability tiers and prices shift; don't assume the last audit's snapshot still holds.

2. **Inventory every "Model Policy" block** across `40_OS/05_Skills/*/SKILL.md` and the scheduled task prompts (list scheduled tasks via the scheduled-tasks tools, or read `40_OS/08_Memory/scheduled-tasks.md`). Delegate this enumeration to a subagent with `model: haiku`... it's a mechanical scan, not a judgment call. Return a flat list: skill/task name, orchestrator tier, subagent delegation (if any), rationale.

   **2a. Compliance sweep (same haiku subagent, same pass).** Extend the enumeration into a compliance check: sweep every skill directory under `40_OS/05_Skills/` (and every scheduled task prompt) for the presence of a "## Model Policy" section. This sweep also runs on `model: haiku`... it's a mechanical presence-check, not a judgment call. Return a compliance list of skills/tasks MISSING the block. For each miss, the haiku subagent classifies the skill's dominant work type (mechanical, moderate, or judgment-heavy, based on the skill's stated purpose) and attaches a suggested policy tiered accordingly: mechanical → Haiku, moderate → Sonnet, judgment → Opus. Missing-block findings feed into the Step 5 proposed-changes table alongside the deprecation/upgrade/downgrade findings, each row noting "Missing Model Policy block" as the reason and the tiered suggestion as the proposed assignment.

3. **Flag deprecated or superseded assignments.** Cross-reference the Step 2 inventory against the Step 1 lineup. Any skill or task assigned a model that's deprecated or has a clearly superior successor at the same tier gets flagged with a proposed replacement.

4. **Flag upgrade and downgrade opportunities.** Upgrade: a higher tier's price has dropped enough (or a tier's quality has risen enough) to justify promoting a task that's currently underserved. Downgrade: a cheaper tier now matches the quality bar for a task currently over-provisioned. Judge against actual observed output quality (ledger evidence, feedback log), not just sticker price.

5. **Output a proposed-changes table** in the audit report, one row per proposed change (skill/task, current assignment, proposed assignment, reason, evidence). Includes the Step 2a compliance findings... skills/tasks missing a Model Policy block, each with its tiered suggested policy. This module never applies model changes unilaterally... every change goes to Zack for approval, same as any other Mercer recommendation under the Authority Model.

6. **Log outcomes in trend tracking.** Append this module's findings (proposed changes, approved/rejected, current tier snapshot) to `40_OS/08_Memory/audit-history.md` so tier drift and model-cost trends are visible month over month, same discipline as the rest of Module 6's metrics.

Canonical tier definitions: `00_Context/os-config.md`, section "Model Selection Policy".

---

## Reference Docs

- `00_Context/os-config.md` ... canonical config (database IDs, emails, paths, rules)

---

## Authority Model

- **Auto-fix:** Move misplaced files, fix minor formatting, update stale timestamps, correct broken cross-references
- **Recommend:** CLAUDE.md restructuring, new skills, deprecations, workflow changes, app features
- **Never:** Delete files without explicit approval. Never restructure the numbered folder hierarchy.

---

## Gotchas

Read `gotchas.md` before every run. Update it when something goes wrong.

---

## Evaluation

Pass the checklist at `eval/checklist.md` before closing the audit.

Your work is being evaluated by two personas. See `eval/advisory-board.md`.

---

## Key Reminders

- Always read persona.md at the start and stay in character throughout
- Auto-fix small stuff, recommend big stuff. Never delete without approval.
- Module 4 is the strategic module... spend the most time here
- Every recommendation needs a tradeoff assessment (effort, payoff, where it lives)
- Trend tracking is mandatory... append to audit-history.md every run
- Use prose for analysis, tables for inventories and comparisons
- Close with the 3 highest-impact recommendations and one "thing I'd build next"
