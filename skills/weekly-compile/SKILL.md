---
name: weekly-compile
description: Weekly full compile pass across uncompiled raw sources in the Obsidian vault. Creates/updates wiki articles, refreshes _index.md, runs integrity check, flags gaps for the week's signal sessions.
---

# Weekly Compile

Full compile pass for the McRay Group LLM Knowledge Base. Runs every Sunday at 7 PM ET to ensure the wiki is current before Monday's Morning Launch.

## Workflow

### Step 1: Read Gotchas
Before starting, read `gotchas.md`. Known failure patterns live there. Aware of these? Proceed.

### Step 2: Orient
Read `instructions/orient.md` for the full orientation sequence.

Load the current state of the knowledge base:
1. Read `40_OS/09_Vault/wiki/_index.md` to get current article count, categories, and open questions.
2. List all files in `40_OS/09_Vault/raw/` and identify sources added or modified since last compile (check file dates or compare against _index.md's "Raw Source Coverage" section).
3. Note the current article count and last_compiled dates across wiki articles.

Log: total raw sources, uncompiled count, current article count.

### Step 3: Classify Uncompiled Sources
Read `instructions/classify.md` for classification rules.

**Unified second brain rule (2026-07-11): every source gets a disposition. "Out of scope" and terminal "skip" are abolished.** The wiki covers all domains, not just the thesis: thesis domains (concepts/, sectors/, people/, companies/, patterns/, evidence/) plus general domains (ai-craft/, business-building/, markets/, playbook/). New domains are created when ~5+ related sources don't fit an existing one. See `40_OS/08_Memory/vault.md` (Wiki Taxonomy + Domain Rules).

For each uncompiled raw source:
1. Read the full content (not just the filename).
2. Determine which existing wiki articles it strengthens (any domain).
3. Identify if it introduces a new concept, person, company, pattern, evidence case, or general-domain topic not yet in the wiki. Company-focused sources compile to `companies/`, human-focused sources to `people/`.
4. Assign exactly one disposition:
   - `compile` ... new or updated article (thesis or general domain)
   - `merge` ... insight folded into an existing article, source backlinked
   - `duplicate` ... same URL/content already captured; add `duplicate-of:` frontmatter, list for delete approval
   - `needs-retrieval` ... truncated/preview-only body; add `needs-retrieval` tag, list in report; do not compile from a preview
   - `archive` ... genuinely no durable value (rare); add `archived` tag so it exits the orphan count

Group sources by disposition. General-domain compiles may use lighter extraction (TLDR + core insight + links); thesis-domain compiles use full extraction per source classification.

### Step 4: Update Existing Articles
Read `instructions/compile.md` for compile rules and quality standards.

For each existing article that has new supporting sources:
1. Read the current article.
2. Integrate new evidence, quotes, or data points into the appropriate sections.
3. Add the raw source path to the YAML `sources:` array.
4. Update `last_compiled:` date to today.
5. Add any new open questions surfaced by the new material.
6. Ensure all outbound links are valid and add new links if the source connects to other articles.

### Step 5: Create New Articles
Read `references/article-standards.md` for the article format spec. This file is the canonical template and format authority for wiki articles.

For each new article candidate:
1. Determine category: thesis domains (concept, sector, person, company, pattern, evidence) or general domains (ai-craft, business-building, markets, playbook). Company-focused sources go to `companies/`, human-focused to `people/`. General-domain articles have no freshness gate and Counter-Arguments is optional (required only for markets/ claims). Companies articles use the distinct frontmatter (type, sector, stage, key_people linking to wiki/people/ entries) per `wiki/companies/_template.md`.
2. Create the article with full YAML frontmatter (title, category, summary, sources, related, last_compiled).
3. Write Core Argument, Evidence, Open Questions, and Links sections.
4. Ensure at least one outbound link and one raw source reference.
5. Save to the appropriate wiki/ subdirectory.

### Step 6: Update _index.md
Read `instructions/index-update.md` for index update rules.

1. Add any new articles to the appropriate category table in `_index.md`.
2. Update summaries for existing articles that were meaningfully changed.
3. Update `article_count` in the YAML frontmatter.
4. Update `last_updated` date.
5. Refresh the "Raw Source Coverage" section with current counts.
6. Review and update "Thesis Pillar Strength" ratings if new evidence shifts confidence.
7. Review and update "Open Question Clusters" if new questions emerged or old ones were resolved.

### Step 7: Integrity Check
Read `instructions/integrity-check.md` for the full check procedure.

Run the weekly integrity check:
1. **Orphan check:** Every wiki article must appear in _index.md. Every _index.md entry must have a corresponding file.
2. **Link check:** Verify all `[[wikilinks]]` in articles resolve to existing articles.
3. **Source check:** Verify all `sources:` paths in YAML frontmatter point to files that exist in raw/.
4. **Freshness check:** Flag any article with `last_compiled` older than 30 days.
5. **Frontmatter check:** Every article must have all required YAML fields (title, category, summary, sources, related, last_compiled).

Log any issues found. Auto-fix what's possible (e.g., missing _index.md entries). Flag the rest for manual review.

### Step 8: Gap Analysis (Biweekly)
Read `instructions/gap-analysis.md` for when and how to run.

On the 1st and 15th of each month (or the nearest Sunday), run the biweekly gap analysis:
1. Which thesis pillars have the thinnest evidence?
2. Are there raw sources that don't map to any wiki article?
3. Which open questions have been open the longest without progress?
4. Are there emerging themes in recent raw sources that deserve new articles?

Save gap analysis findings to the report.

### Step 9: Connection Discovery (Monthly)
Read `instructions/connection-discovery.md` for when and how to run.

On the 1st of each month (or the nearest Sunday), run connection discovery:
1. Read all wiki articles and identify potential cross-links not yet made.
2. Look for patterns across categories (e.g., a person article that should link to a new pattern, or a company article missing key_people links).
3. Suggest new connections and add them if they're clearly valid.

### Step 10: Outputs Hygiene (Promote-or-Prune)
Scan `40_OS/09_Vault/outputs/` for files older than 7 days (by file date). Classify each:

1. **Automation/run logs** (vault-linker reports, vault-lint reports, weekly-compile reports, sync artifacts, any machine-generated run output): move to `40_OS/09_Vault/outputs/_archive/YYYY-MM/` (current month; create the folder if it doesn't exist).
2. **Substantive synthesis notes** (session outputs, hagen reframes, luce drafts, anything carrying original thinking): do NOT move. List them in the compile report as promotion candidates for Zack's call... promote to `raw/` (so they compile into the wiki) or explicitly retain in outputs/.

Never delete anything in this step. Skip `_archive/` itself and active work subfolders (e.g. `luce/`)... list their stale contents as promotion candidates instead of moving them. When in doubt whether a file is a log or a synthesis note, treat it as substantive and list it.

### Step 11: Report
Read `instructions/reporting.md` for report format.

Generate a compile report:
- ET timestamp
- Raw sources processed / skipped / total
- Articles updated (list with brief description of changes)
- Articles created (list with category and summary)
- Integrity check results (pass/fail, issues found)
- Gap analysis findings (if biweekly run)
- Connection discovery findings (if monthly run)
- Outputs hygiene: N files archived to `_archive/YYYY-MM/`, plus the promotion-candidate list (filename + one line each)
- Recommendations for this week's signal sessions

### Step 12: Verify
Run `eval/checklist.md` to verify all steps completed correctly.

## Rules

- No em dashes anywhere in generated files. Use commas, periods, semicolons, or "..." instead.
- Use ET timestamps always.
- Never delete raw sources. They are the permanent record.
- Never overwrite wiki articles without reading them first. Always integrate, don't replace.
- When in doubt about which domain a raw source belongs to, compile it lightly into the closest general domain rather than leaving it undispositioned. Only ambiguity about factual accuracy warrants a flag for manual review.
- Freshness check applies to thesis-domain articles only. General-domain articles are never stale.
- The _index.md must always reflect the true state of wiki/. If there's a discrepancy, _index.md is wrong.
- Every new article must have at least one outbound link and one raw source reference. Orphans are bugs.

## Key Files

| File | Purpose |
|------|---------|
| gotchas.md | Known failure patterns, read first |
| instructions/orient.md | Orientation and state loading |
| instructions/classify.md | Raw source classification rules |
| instructions/compile.md | Compile rules and quality standards |
| instructions/index-update.md | _index.md update procedure |
| instructions/integrity-check.md | Weekly integrity check procedure |
| instructions/gap-analysis.md | Biweekly gap analysis procedure |
| instructions/connection-discovery.md | Monthly connection discovery procedure |
| instructions/reporting.md | Report format and content |
| references/article-standards.md | Wiki article format spec |
| eval/checklist.md | Pass/fail tests before completing |
