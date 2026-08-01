---
name: Retrieve steering patterns and submit a trace
description: Pre-task, fetch reasoning patterns (E-traces) to inject into the system prompt, score each step's difficulty, then submit the completed run for distillation — the ReasonBlocks three-call harness integration.
api: openapi/reasonblocks-openapi-original.json
operations:
- retrieve_traces_v1_traces_retrieve_post
- score_step_v1_score_post
- store_trace_v1_traces_post
---

# Retrieve steering patterns and submit a trace

Use this to wire ReasonBlocks into any custom agent loop over HTTP (no SDK). Base URL
`https://rb-api.reasonblocks.com`; every request carries `Authorization: Bearer <rb_live_… or rb_test_… key>`
and `Content-Type: application/json`.

## Steps

1. **Pre-task — retrieve patterns.** Call `retrieve_traces` (`POST /v1/traces/retrieve`) with the task
   `context`, a `tier` (`e1` instance-level, `e2` failure-mode, `e3` universal), and `top_k`. Inject the
   returned patterns into your agent's system message. An empty `traces` array is valid — it means no
   pattern matched or you are over the monthly intervention cap.
2. **Per-step — score difficulty (optional).** Call `score_step` (`POST /v1/score`) to get the heuristic
   FSM difficulty state for the current step; use it to route easy steps to a cheaper model and escalate
   hard ones.
3. **Post-task — submit the trace.** After the run finishes, call `store_trace` (`POST /v1/traces`) with
   the completed trace so ReasonBlocks distills it into future steering patterns.

## Rules

- **Auth & tenancy:** the key's `org_id`/`project_id` scope is authoritative; body `org_id` fields are ignored.
- **No idempotency keys.** Retries are not deduplicated server-side — design writes to be safe on retry.
- **Rate limits:** 200 req/sec per key (400 burst); on `429` honor `Retry-After: 1`.
- **Errors:** bodies are `{"detail": "<reason>"}`. `401` invalid key, `422` schema violation, `5xx` retry with backoff.
- Target `/v1/` paths; unversioned aliases are legacy SDK back-compat only.
