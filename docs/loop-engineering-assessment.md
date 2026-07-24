# Loop Engineering Assessment: Zack's OS Against the 2026 Frameworks

Date: 2026-07-13
Inputs: full vault read (5 raw entries + Agent Harness Engineering wiki article), 3 previously uncaptured sources retrieved in full (Codez 14-step loop engineering article, Osman /goal prompt, Walker validation-agent prompt), and a workflow inventory sweep of 40_OS (loops-map.md, scheduled-tasks, 10 central skills).

## The One-Line Read

The OS is a strong harness-and-memory system with real chains, but almost nothing in it iterates toward a goal condition within a run; the loop layer... the fourth rung the Anthropic quote names... is the gap, and the fastest wins are in `~/Developer`, not the Work folder.

## The 4 Assessment Frameworks

**1. The four-layer stack: agent > harness > loops > memory (+ evals).** A modern agent has all four layers. The retrieved memory source extends it: memory + loops + harness + evals = self-improving agent system.

**2. The operator ladder: prompt > skill > chain > plugin > loop.** Englert's ladder plus the Codez fourth rung. A loop is a system that finds the work, dispatches it, verifies the result, records state, and decides the next move without a human prompting each cycle.

**3. Your own three-level test: see the loop > close the loop > make it recursive.** From your 2026-04-07 thesis note. Level 3 is a loop that generates data about its own failures and feeds that back into its design.

**4. The loop qualification and safety rules (Codez/Osmani, retrieved in full).** The 4-condition test before building: task repeats weekly+, verification is automated, token budget absorbs waste, agent has logs and a repro environment. Plus the build rules: objective gate separate from the maker, state file on disk paired with a standing spec, hard stops, cost per accepted change above 50%, and the Ralph Wiggum failure (soft "done" conditions let the loop exit half-finished).

## Scorecard: Where Each Workflow Sits

| Workflow | Ladder rung | Stack layers present | Your 3-level test | Verdict |
|---|---|---|---|---|
| memory-sync (overnight) | Loop (genuine) | harness + memory + loop; no eval | Level 3, recursive | Your best loop. Decay > dedupe > pattern-detect > rehearsal is real self-improvement with a 200-line budget cap. |
| advisor | Chain, goal-shaped | harness + memory (commitments.md with aging) | Level 2.5 | Persistent state across days, escalation on staleness. Closest judgment loop; still one pass per evening. |
| mercer | Chain, system-level | harness + memory (audit-history trends) | Level 2.5 | Audits the OS but nothing audits mercer, and audit-history is written but never read for compounding insight (your own loops-map gap #15). |
| begin-the-day / end-of-day / weekly-review | Chain | harness + memory | Level 2 | Real multi-timescale feedback (EOD writes what AM reads, Friday plans the week) but each run is a fixed pass, no within-run iteration. |
| day-ledger | Chain | harness + memory | Level 2 | Two-way Roadmap loop closes state, single pass. |
| flag-that > weekly rollup > gotchas | Loop, latency-bound | harness + memory; weak eval | Level 2.5 | A real improvement loop that closes only weekly. Capture is instant, closure waits up to 7 days. |
| pulse | Skill | harness | Level 1 | Fine as-is; read-only detection with "silence is the signal." |
| Coding work in ~/Developer | Prompt/skill | agent only | Level 1 | The biggest gap. No /goal loops, no standing validation agents, despite this being where automated verification already exists. |

## Best Practices You Should Adopt (from the retrieved sources)

**1. Write goals as contracts, not wishes.** Every /goal needs: outcome (what's true when done), verification surface (the artifact that proves it), constraints, boundaries, iteration policy, and exit criteria for the blocked case. Walker's 6-phase prompt is the working template: per-phase exit criteria, a canonical spreadsheet as state, and "never declare completion unless all exit criteria are satisfied." Full text now in the vault.

**2. Separate maker from checker, including at the stop condition.** The model that wrote the code grades its own homework too kindly. Anthropic's evaluator-optimizer pattern; Codex /goal implements it by having a separate small model check completion. Any loop you build gets an objective gate (tests, typecheck, build), not a second opinion.

**3. Run the 4-condition test before building any loop.** Most candidates fail it, and a single well-aimed prompt wins. Your OS skills mostly fail condition (b)... their outputs are judgment calls, not automated verifications... which is why they're correctly built as chains, not loops. Your repos pass all 4 conditions once they carry test suites.

**4. State on disk, spec reread every run.** You already do this well (tomorrow-focus.md, commitments.md, next-week-plan.md, active-memory.md). The addition: pair each loop's state file with a standing spec so long runs don't drift... "state tells the agent where it is, the spec tells it where to go."

**5. Hard stops and one metric.** Token budget, iteration cap, or time limit on every loop, human gate before anything irreversible, and track cost per accepted change; below 50% acceptance the loop is losing. This is the thin defensible slice your harness article already names (loop guardrails, budget caps, scheduler wiring).

**6. Minimum viable loop, in order.** One automation + one skill + one state file + one gate. Reliable manual run first, then skill, then loop, then schedule. Good first loops: CI triage, dependency bumps, lint-and-fix, flaky-test repro. Keep loops off architecture, auth, payments, and judgment-call work.

**7. Treat unattended loops as attack surface.** Security scans inside the gate, audit third-party skills (520 of 17,022 audited community skills leak credentials), sanitize logs, re-audit permissions every 30 days.

## Recommended Moves, Ranked

1. **First rep: a Walker-style /goal validation loop on one repo in ~/Developer.** Pick the repo with the best test coverage. This is where the 4-condition test passes today and where the practice reps live. Adapt the Phase 1-6 template; add a token budget and iteration cap before running unattended.
2. **Close loops-map gap #15: make mercer recursive.** Add a module (or a small companion skill) that reads audit-history.md for compounding trends and audits mercer's own hit rate on past recommendations. This upgrades your strongest system-level chain to Level 3.
3. **Tighten the flag-that loop's latency.** The weekly rollup is the only closure mechanism. Option: let memory-sync's daily pass promote high-signal flags into gotchas proposals same-day instead of waiting for Friday.
4. **Add the missing eval layer to memory-sync.** It self-improves but nothing measures whether memory quality is actually rising. A simple monthly eval (does active-memory.md correctly answer 10 spot-check questions from the ledgers?) closes the formula: memory + loops + harness + evals.
5. **Codify a loop-guardrail standard.** One page in 00_Context or working-rules: every future loop declares its gate, state file, spec, budget cap, and acceptance metric. This is the portable, defensible slice the harness commoditization scan said to build.
6. **Retrieval leftover:** the Codez "12-step Claude agent memory" article is still uncaptured and flagged in the vault as a target.

## What Not to Do

Don't loop-ify the daily OS skills. Begin-the-day, end-of-day, and advisor produce judgment outputs with no automated verifier, so they fail condition (b) of the qualification test; forcing them into loops would burn tokens verifying the unverifiable. They're correctly architected as chains with human gates. The loop practice belongs in the repos, and later in portco harnesses where processes have measurable gates (invoices reconciled, tickets closed, quotes sent).

## Vault Updates Made Alongside This Assessment

- `raw/2026-06-22-loop-automation-inside-codex-goal-prompt.md` ... full Osman prompt captured, When To Use / Build Notes filled.
- `raw/2026-06-22-goal-continuous-software-quality-validation-prompt.md` ... full Walker 6-phase prompt captured verbatim.
- `raw/2026-07-01-codez-build-memory-self-improving-agents.md` ... memory taxonomy (procedural/semantic/episodic) and the memory+loops+harness+evals formula captured.
- `raw/2026-06-29-codez-anthropic-agents-lead-self-improving-loops.md` ... full 14-step roadmap appended, with a provenance note that the ">90%" figure is likely engagement-inflated while the mechanism claim is corroborated (Boris Cherny, Anthropic's own material).

Key external sources: Addy Osmani "Loop Engineering" (addyosmani.com/blog/loop-engineering), Codez 14-step article (Rattibha archive), OpenAI "Using Goals in Codex" cookbook, Anthropic "Building Effective Agents."
