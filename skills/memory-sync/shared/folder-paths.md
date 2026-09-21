# Folder Paths

Refreshed 2026-07-27. Previous version (2026-04-09) was stale: it described the retired Obsidian vault layout (Inbox/, Thesis/, Signal/, Reference/, Sessions/, Sectors/, People/, Templates/), which was archived on 2026-04-09 and replaced by the compile-forward architecture below.

**This file is deliberately thin.** Where to save the common things, and pointers for everything else.

| For | Read |
|---|---|
| Full folder map, data flows, naming conventions | `00_Context/os-architecture.md` |
| Vault architecture in detail | `40_OS/08_Memory/vault.md` |
| Config values (IDs, emails, paths) | `00_Context/os-config.md` |

---

## Top Level

`00_Context/` · `01-mcray-group/` · `20_Career/` · `30_Projects/` · `40_OS/` · `99_Archive/`

`01-mcray-group` is the grandfathered kebab-case exception to the `NN_Title` convention. Everything else follows it.

## 40_OS Interior

`01_Workflows` · `02_Agents` · `03_Automations` · `04_Prompts` · `05_Skills` · `06_Templates` · `07_Feedback` · `08_Memory` · `09_Vault` · `10_Migrations` · `11_Best Practices` · `12_Call_Prep` · `13_Dashboards` · `14_Plugins` · `15_Tooling` · `16_Handoffs`

Dashboards, Plugins, and Tooling were renumbered from 06/07/10 to 13/14/15 in the July 2026 reorg. If you find a doc citing `40_OS/06_Dashboards`, `40_OS/07_Plugins`, or `40_OS/10_Tooling`, it is stale.

## Where to Save

| Thing | Path |
|---|---|
| Memory (living file) | `40_OS/08_Memory/active-memory.md` |
| Memory archives | `40_OS/08_Memory/memory-archive-YYYY-MM.md` |
| People profiles | `40_OS/08_Memory/people/` (kebab-case filenames) |
| People quick reference | `40_OS/08_Memory/people-quickref.md` |
| Glossary | `40_OS/08_Memory/glossary.md` |
| Daily ledgers | `40_OS/08_Memory/daily-ledger/YYYY-MM-DD.md` |
| Call prep docs | `40_OS/12_Call_Prep/` (archive: `_archive/YYYY/`) |
| Skills (editable source) | `40_OS/05_Skills/[skill-name]/` |
| Skill packages (.skill) | `40_OS/05_Skills/dot_skills/` |
| Component docs | `40_OS/01_Workflows/01_Component Docs/` (.docx) |
| Templates | `40_OS/06_Templates/` |
| Handoffs and bridge docs | `40_OS/16_Handoffs/` (never `00_Context/`) |
| Changelog | `40_OS/CHANGELOG.md` |

## Personal Thinking Vault

`40_OS/09_Vault/`. Compile-forward, tool-agnostic (Obsidian is just the current viewer):

- **`raw/`** ... immutable sources. Captures land here and are never edited after the fact.
- **`wiki/`** ... the LLM-compiled knowledge base. Start every vault session by reading `wiki/_index.md`. Subfolders: ai-craft, business-building, companies, concepts, evidence, health, markets, patterns, people, playbook, sectors.
- **`outputs/`** ... ephemeral session work (`outputs/luce/` for Luce pieces).
- **`research/`** ... per-topic research folders.

The legacy folders are archived at `09_Vault/_archive_2026-04-09/`. There is **no** live `Inbox/` or `Templates/` at the vault root; capture goes straight to `raw/`.

## McRay Group Company Vault

`01-mcray-group/50-vault/`. Institutional knowledge for the firm, separate from the personal thinking vault. **Five** subfolders: `identity/`, `thinking/`, `decisions/`, `sops/`, `artifacts/`.

(Note: CLAUDE.md and os-config.md both describe this as a "six-subfolder architecture" while naming five. Five is what exists on disk. Flagged 2026-07-27, not yet corrected at the source.)

## Code

**Code repos live in `~/Developer/[repo-name]`, never inside the Work folder or any iCloud path.** iCloud evicts `.git` and `node_modules` and breaks builds. Product build-docs (PRDs, specs) live in-repo under `docs/`. Cowork cannot reach `~/Developer` ... prepare Terminal commands for Zack instead. Quick path lookup: `00_Context/code-locations.md`.

Non-code projects (learning, research) live in `30_Projects/`.
