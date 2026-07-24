# Surfaces and Trust Boundaries

The OS runs across 3 surfaces, each with its own trust level. The split exists because "an AI assistant" is not one thing: an attended session with a human watching every action can safely hold high-trust credentials, while an autonomous agent processing untrusted input unattended cannot. Capability follows supervision.

## The 3 surfaces

**Cowork (attended knowledge work).** Full filesystem access to the knowledge base, high-trust credentials, service-role database access. Justified on one premise: a human is present for every action.

**Claude Code (attended coding).** Full access to code repos, attended terminal sessions. Same supervision premise, different domain.

**Argus (autonomous operations).** The always-on Chief of Staff agent. Runs headless on a hardened remote box with no filesystem access to the knowledge base. Deliberately low-trust, because it reads untrusted input all day... inbound email bodies, meeting transcripts, Slack messages... with nobody watching. Anything in an autonomous agent's context or reach can be exfiltrated or poisoned through crafted input, so Argus interacts with the rest of the system only through narrow, individually granted capabilities.

## Trust principles

**Staged privilege.** Argus's capabilities are granted one at a time, each gated on observed clean behavior rather than granted at launch. Counterintuitively, writes are granted before reads: a quarantined write can be reviewed before it does harm, while a read leaks its contents immediately through the agent's outputs. A prompt-injected agent with semantic recall over memory can be steered to query and leak that memory through a draft email... so semantic read is the higher-risk grant even though it feels benign.

**RPC-only surface, no table grants.** Argus's database role has zero table grants. It gets EXECUTE on a handful of SECURITY DEFINER functions with pinned search paths, input validation in SQL (not in the agent's prompt), and provenance hardcoded inside the function so the agent cannot spoof its own source tag. Full pattern in `memory-substrate-pattern.md`.

**Context packs, not context access.** Argus never receives raw profile documents. It gets a curated bundle of distilled behavior instructions and need-to-know facts under an explicit allowlist manifest. The operating principle: anything in an agent's context can be exfiltrated, so the context contains only what the job requires.

**Quarantine-then-file.** Captures written by Argus land as pending rows in the signal store and stay there until a human-attended drain loop reviews and files them into the canonical knowledge base. Autonomous input never writes canonical memory directly.

**Mutual watchdog.** The attended and autonomous surfaces monitor each other through a shared heartbeat table. The Mac's morning and overnight passes flag if Argus's hourly beat goes stale past 24 hours; Argus alerts if the Mac's daily beats stop. Neither side assumes the other is healthy.

## Why this design

Most personal-agent setups either give the agent everything (and inherit the full blast radius of a prompt injection) or give it nothing (and lose the value of autonomy). The surface split takes a third path: supervision determines trust, trust determines capability, and every capability granted to the unattended surface is individually reviewed, quarantined where possible, and revocable in one line of SQL.
