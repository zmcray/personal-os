# Skill Conventions

Zack-specific conventions that layer on top of the skill-creator plugin. skill-creator is the authority for how to build, test, and iterate on skills. This file only adds local rules.

---

## Folder Layout: Two Platforms

This folder holds AI workflow extensions for two platforms:

- **Cowork skills** ... folder-per-skill at `40_OS/05_Skills/[skill-name]/`. Packaged to `.skill` files, mounted by Cowork. Everything else in this doc applies to these.
- **Claude Code slash commands** ... flat `.md` files in the `~/Developer/dev-workflow` repo under `commands/` (moved out of the Work folder 2026-06-13; version-controlled, edited in CC). Live install lives at `~/.claude/commands/`. Native Claude Code format (frontmatter + markdown body). Not packaged, not mounted. The same repo holds the canonical build-workflow block (`AGENTS.workflow.md`) and the AGENTS.md/CLAUDE.md templates; `deploy-agents-md.sh` pushes those into every repo under `~/Developer`. See `~/Developer/dev-workflow/README.md`.

Sync Claude Code commands to live install:
```bash
cp ~/Developer/dev-workflow/commands/*.md ~/.claude/commands/
```

The conventions below (3-Step Gate, Tiered Structure, SKILL.md Principles, Eval Infrastructure) apply to Cowork skills only.

---

## The 3-Step Completion Gate

Every skill create or update must complete all 3 steps. Do not stop after editing files. Do not ask whether to do them.

1. **Edit the skill folder** in `40_OS/05_Skills/[skill-name]/`. This is the writable backup. `.claude/skills/` is read-only ... edits there silently fail.

2. **Package + present the .skill file.** This is the only way to update the live skill. The skill is not updated until Zack clicks "Save skill."
   ```bash
   cd /mnt/Work/40_OS/05_Skills/[skill-name] && zip -r /sessions/$(basename $(pwd -P | sed 's|/mnt/Work.*||'))/[skill-name].skill . -x ".*"
   ```
   Then: save a copy to `40_OS/05_Skills/dot_skills/`, and call `present_files` with the .skill file.

3. **Add a CHANGELOG entry.** One line in `40_OS/CHANGELOG.md` with date, skill name, and what changed.

doc-that is optional ... only when explicitly requested.

---

## Tiered Skill Structure

Use the simplest structure that produces reliable output.

### Tier 1: Capture Skills
One-shot database writes or log entries. No judgment, no voice sensitivity.

```
skill-name/
├── SKILL.md       Orchestrator (can be self-contained if < 100 lines)
└── gotchas.md     Known failure patterns
```

Examples: task, thesis, buildnote, flag-that, strategy

### Tier 2: Workflow Skills
Multi-step workflows with predictable output.

```
skill-name/
├── SKILL.md           Thin orchestrator pointing to instruction files
├── gotchas.md         Known failure patterns
├── instructions/      One file per distinct concern
└── templates/         Output format templates (only if non-obvious)
```

Examples: folder-cleanup, content-performance, signal-classify-sync, mercer, vault-linker

### Tier 3: Full Skills
Output quality depends on voice, judgment, or creative nuance.

```
skill-name/
├── SKILL.md           Thin orchestrator, just pointers
├── gotchas.md         Known failure patterns
├── instructions/      One file per distinct concern
├── references/        Schemas, profiles, detailed material
└── templates/         Output format templates and guides
```

Examples: content-engine, content-reply, hagen, luce, email-draft, signal-session, post-call-debrief, begin-the-day, end-of-day, weekly-review, doc-that, smb-job-search

### Deciding the Tier

1. One-shot database write or log entry? → Tier 1
2. Multi-step process with predictable output? → Tier 2
3. Voice, judgment, or creative nuance? → Tier 3
4. Unsure? Start at Tier 1 or 2. You can always add structure later.

---

## Model Policy (required in every skill)

Every new or updated SKILL.md must include a "## Model Policy" block near the top declaring:

- **Orchestrator tier** for headless/scheduled runs (Haiku, Sonnet, or Opus... never Fable headless).
- **Subagent delegation** with explicit `model:` params for volume work.
- **No-escalation rule.**

Attended runs inherit the session model unless stated. Canonical tiers: `00_Context/os-config.md` "Model Selection Policy". A skill update is incomplete without this block.

---

## SKILL.md Principles

- SKILL.md is the orchestrator, not the encyclopedia
- Tier 1: self-contained is fine
- Tier 2-3: thin orchestrator with pointers to instruction files
- Keep under 500 lines
- Reference files clearly with guidance on when to read them

---

## Eval Infrastructure

Only include `eval/` files in skills you are actively benchmarking via skill-creator. For most skills, gotchas.md and production feedback (flag-that) are sufficient quality controls.
