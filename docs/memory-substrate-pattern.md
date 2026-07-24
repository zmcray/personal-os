---
title: "Shared memory substrate for multi-surface agents (two-store, RPC-only, staged privilege)"
date: 2026-07-22
category: architecture-patterns
module: argus
problem_type: architecture_pattern
component: database
severity: medium
applies_when:
  - "multiple agent surfaces (attended and autonomous) must feed one shared memory store"
  - "canonical knowledge must remain a human-readable markdown file, not a database"
  - "a low-trust autonomous agent needs database write access without table grants"
  - "embedding generation must not block or complicate writers"
  - "target schema is not PostgREST-exposed and needs helper RPC bridges"
related_components:
  - assistant
  - background_job
  - documentation
tags:
  - supabase
  - pgvector
  - pg-cron
  - security-definer
  - edge-functions
  - least-privilege
  - embeddings
  - memory-architecture
---

# Shared memory substrate for multi-surface agents (two-store, RPC-only, staged privilege)

## Context

The OS has two agent surfaces that both generate memory raw material:

1. **Cowork** ... an attended assistant with filesystem access to the Work folder, running with service-role credentials.
2. **Argus** ... an autonomous, headless CoS agent on a VPS with **no filesystem access** to the Work folder and a deliberately low-trust posture (it reads untrusted input all day: inbound email bodies, meeting transcripts, Slack).

The canonical memory is a human-curated markdown file (`active-memory.md`), written by exactly one consolidation skill (memory-sync). The gap: Argus had nowhere durable to put thinking captures or observations, and any naive fix (giving the agent DB tables, or shipping context files to the box) collides with the injection threat model... anything a compromised agent can read is exfiltratable into a draft or CRM note via crafted input, and anything it can write freely can poison downstream memory.

Built and verified live 2026-07-22 on Supabase project `<SUPABASE_PROJECT_ID>` (migrations `os_memory_layer`, `memory_embed_surface`; edge function `embed-observations`).

## Guidance

The pattern decomposes into seven pieces. Each is independently reusable.

### 1. Two-store model with single-writer discipline

Split memory into two stores with one-directional flow:

- **Signal/record store (Supabase, `os` schema):** raw rows, append-friendly, machine-written by any surface.
  - `os.captures` ... kind-typed thinking captures with a lifecycle: pending → filed (or dropped), plus `filed_to` provenance (the vault path each capture landed at) and `source` (argus/cowork/claude-code/zack).
  - `os.memory_observations` ... append-only raw material. No drain lifecycle; consolidated, never deleted by the loop. Columns include `embedding public.vector(1536)` nullable, `embed_model` text, HNSW cosine index.
- **Narrative store (markdown, `active-memory.md`):** human-curated canonical memory. Written by **exactly one** consolidation skill (memory-sync), which gathers recent observation rows and distills judgment into prose.

Rows flow Supabase → markdown, never the reverse. Multiple writers contribute raw material; one writer owns the canonical artifact. This eliminates merge conflicts, provenance ambiguity, and "which copy is true" questions entirely.

### 2. RPC-only agent surface (no table grants)

The low-privilege role `argus_agent` has **zero table grants**. It gets EXECUTE only, on SECURITY DEFINER functions in `public` with a pinned `search_path` and input validation:

```sql
create or replace function public.argus_capture(p_kind text, p_text text)
returns uuid language plpgsql security definer
set search_path to 'os','public' as $$
declare v_id uuid;
begin
  if p_text is null or length(trim(p_text)) = 0 then raise exception 'empty capture'; end if;
  if p_kind not in ('thesis','strategy','brand','learn','other') then raise exception 'invalid kind: %', p_kind; end if;
  insert into os.captures (kind, text, source) values (p_kind, trim(p_text), 'argus') returning id into v_id;
  return v_id;
end $$;
revoke all on function public.argus_capture(text,text) from public, anon, authenticated;
grant execute on function public.argus_capture(text,text) to service_role, argus_agent;
```

Notes on the shape: RLS is on with no policies (belt and suspenders under SECURITY DEFINER); `source` is hardcoded to `'argus'` inside the function so the agent cannot spoof provenance; the kind whitelist and empty-text check live in SQL, not in the agent's prompt.

### 3. Staged privilege: write-before-read, grant-on-evidence

Privileges land in stages tied to observed behavior, and the code is written so each future grant is trivially small:

- `argus_capture` ... granted to the agent **now** (lowest-risk write; rows are quarantined as pending until a human-supervised drain loop files them).
- `argus_observe(text)` ... **created in the same migration** but granted only to `service_role`. The agent's grant is deferred until roughly a week of clean live operation, at which point it is a logged one-liner.
- Semantic **read** (`match_memories`) ... withheld from the agent entirely, pending an explicit threat review. Retrievable memory is an exfiltration surface: a prompt-injected agent with semantic recall can be steered to query and leak memory contents through a draft.

The asymmetry is deliberate: agent **writes** are quarantined by lifecycle; agent **reads** leak immediately, so read is the higher-risk grant even though it feels benign.

### 4. Dumb writers + async embedding backfill

Writers never compute embeddings. Rows insert with `embedding` null; an async worker backfills:

- pg_cron every 15 min → `public.memory_embed_now()` (SECURITY DEFINER wrapper around `net.http_post`, calling the edge function with the publishable anon JWT, `verify_jwt` on)
- Edge function `embed-observations` (backfill mode): reads batches of up to 100 via helper RPC `memory_rows_to_embed(p_limit)`, calls OpenAI `text-embedding-3-small` (1536 dims), writes back via `memory_set_embedding(p_id, p_embedding, p_model)`, stamping `embed_model` on every row.

This keeps every write path (agent RPC, Cowork skill, manual SQL) free of API keys, latency, and failure coupling. An embedding outage degrades to "rows queue with null embeddings" instead of "writes fail." The model stamp means a future model migration can re-embed selectively.

### 5. Helper-RPC bridge for unexposed schemas

The `os` schema is **not** exposed through PostgREST, so the edge function cannot query it directly. The bridge is SECURITY DEFINER helper functions in `public`, granted to `service_role` only (`argus_agent` gets nothing): `memory_rows_to_embed`, `memory_set_embedding`, `match_memories`, `memory_embed_now`.

A detail that matters: the write-back RPC takes the embedding as **text** and casts inside SQL, which is more robust through PostgREST than a vector-typed parameter:

```typescript
// edge function: pass the embedding as a JSON-array string, not a vector param
await rpc("memory_set_embedding", {
  p_id: row.id,
  p_embedding: JSON.stringify(embedding), // text param
  p_model: "text-embedding-3-small",
});
```

```sql
-- inside memory_set_embedding: cast text to vector
update os.memory_observations
set embedding = p_embedding::public.vector, embed_model = p_model
where id = p_id and embedding is null;
```

### 6. Graceful degradation as a testing strategy

If `OPENAI_API_KEY` is unset, the edge function returns 503 with a clear message and rows queue harmlessly. This is not just resilience... it enabled **smoke-testing the entire plumbing before the key existed**: cron → pg_net → JWT auth → function invocation was verified by asserting the graceful 503. The dependency (the API key) was decoupled from the infrastructure verification.

### 7. In-function role gating for dual-mode endpoints

The same edge function serves search mode (`POST {query, k}` → embed query → `match_memories(p_query_embedding, p_k)`, cosine similarity `1 - (embedding <=> qe)`). The gateway verifies the JWT signature; the function then decodes the (already-verified) JWT's **role claim** and rejects anything that is not service-role. Verified: an anon-credential search call returns 403. One deployed function, two trust levels, gate enforced where the sensitive capability lives.

### Gotchas (all hit live)

1. **zsh line-wrap ate a CLI flag.** `npx supabase secrets set KEY=... --project-ref <ref>` pasted with a line wrap: the `--project-ref` fragment executed as its own command (`zsh: no such file or directory`) and the secret **silently never landed**. Fixed via the dashboard secrets page. When diagnosing secret visibility, redeploy the function (new version) to force a fresh env read.
2. **Supabase MCP `execute_sql` returns only the last statement's result set.** Multi-statement scripts hide earlier results; split verification queries into separate calls.
3. **`net.http_post` is async... check the response by request id.** Query `net._http_response` by the returned request id, never `order by id desc limit 1`. An unattended cron tick fired mid-verification and nearly got mistaken for the manual test's response.
4. **pgvector `vector`, pg_cron, and pg_net were available-but-not-installed** extensions; enable them explicitly in the migration.

## Why This Matters

Each piece protects against a specific failure:

- **Two-store + single-writer** protects the canonical memory from machine noise and write contention. Raw signal accumulates cheaply; judgment is applied once, by one curator, in one place. Provenance (`source`, `filed_to`) survives the whole pipeline.
- **RPC-only surface** protects against a compromised or confused agent. The agent physically cannot SELECT, UPDATE, DELETE, spoof `source`, or invent a `kind`... the blast radius of full agent compromise is "inserted some pending rows a human reviews."
- **Staged privilege, write-before-read** turns trust into an earned, observable, one-line-diff progression instead of a day-one grant. Withholding semantic read acknowledges that for an agent processing untrusted input, *retrieval is the exfiltration channel*... the threat review happens before the grant, not after the incident.
- **Dumb writers + async backfill** keeps writes fast, key-free, and failure-isolated.
- **Helper-RPC bridge** lets you keep the real schema off PostgREST (smaller attack surface) while still serving edge functions.
- **Graceful degradation** converts "blocked on a secret" into "verified everything except the secret."
- **In-function role gating** avoids deploying two functions or exposing search to publishable keys.

## When to Apply

- Two or more agent surfaces (attended + autonomous, or multiple autonomous) need to contribute to one shared memory or knowledge store.
- A human-curated artifact is canonical and must stay so; machine writers feed it but never touch it.
- At least one writer is low-trust or autonomous... headless, exposed to untrusted input (email, transcripts, web), or new enough that its behavior is unproven.
- The prompt-injection threat model is live: the agent's context or retrievable memory could be exfiltrated through its outputs (drafts, notes, messages).
- You are on Supabase/Postgres with pgvector and want semantic recall without coupling writers to an embedding API.
- Your real tables live in a schema not exposed through PostgREST and edge functions need access.
- You want to stage capability grants on live evidence rather than granting everything at launch.

Skip it when there is a single trusted writer, no autonomous surface, and no injection exposure... a plain table with service-role access is fine.

## Examples

**1. Staged grant, before/after.** The migration ships both functions; only the trust level differs, and promotion is a one-liner:

```sql
-- at migration time (day 0): function exists, agent cannot call it
revoke all on function public.argus_observe(text) from public, anon, authenticated;
grant execute on function public.argus_observe(text) to service_role;

-- after ~1 week of clean live operation (logged in the plan doc):
grant execute on function public.argus_observe(text) to argus_agent;
```

**2. Async HTTP verification, wrong vs right.** `net.http_post` returns immediately with a request id; the response lands later in `net._http_response`.

```sql
-- WRONG: races with unattended cron ticks (one fired mid-verification)
select * from net._http_response order by id desc limit 1;

-- RIGHT: capture the request id from the trigger call, then check exactly that response
select public.memory_embed_now();          -- returns the pg_net request id
select status_code, content
from net._http_response
where id = <returned_request_id>;
```

**3. End-to-end recall verification.** Seed → wait for backfill → self-similarity search must return the seed at similarity ~1.0 (the anon-rejection assertion is covered by the live-run log below):

```sql
-- confirm the seed row got embedded and stamped
select id, embed_model, embedding is not null as embedded
from os.memory_observations
order by created_at desc limit 1;

-- recall proof without an embeddings key: use a stored embedding as its own query
select m.similarity, left(m.text, 60)
from os.memory_observations o,
     lateral public.match_memories(o.embedding::text, 3) m;
```

Live run results: seed row embedded with `text-embedding-3-small` stamp; self-similarity recall = 1.0; anon search 403; unattended cron tick observed returning `embedded:0` on an empty queue.

## Related

- `docs/plans/2026-07-22-002-os-memory-layer.md` ... plan of record for this substrate (design, migrations, verification log, staged-grant sequencing).
- `docs/plans/2026-07-22-001-argus-context-pack.md` ... the same threat model applied to the *inbound* direction: context shipped to the low-trust box is distilled behavior instructions under an explicit manifest (never raw profile files, no globbing), because anything in the agent's context is exfiltratable through crafted input. Substrate and context pack are the two halves of one trust boundary.
- `40_OS/16_Handoffs/memory-layer-track-c-handoff.md` ... execution handoff covering the staged grants and cutover gating.
- `phase-2-deploy/phase-2-spec.md` ... decision P2-7 (direct cutover, guardrails retained) that sets the "clean live operation" gate the staged grant hangs on.
