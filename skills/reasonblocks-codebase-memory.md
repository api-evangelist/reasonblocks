---
name: Persist and recall codebase findings
description: Store per-codebase agent findings, semantically search them before new work, and invalidate stale findings by changed-file blast radius — the ReasonBlocks CodebaseMemory flow.
api: openapi/reasonblocks-openapi-original.json
operations:
- store_finding_v1_findings__codebase_id__post
- search_findings_v1_findings__codebase_id__search_post
- invalidate_findings_v1_findings__codebase_id__invalidate_post
---

# Persist and recall codebase findings

Give a coding agent durable memory of how it solved problems in a repository, so it spends less time
rediscovering the same facts. All routes are namespaced by `codebase_id`. Base URL
`https://rb-api.reasonblocks.com`, bearer auth.

## Steps

1. **Recall before working.** Call `search_findings` (`POST /v1/findings/{codebase_id}/search`) with the
   current task context to semantically retrieve prior findings for this codebase; fold hits into the
   agent's context.
2. **Store new findings.** When the agent learns something durable, call `store_finding`
   (`POST /v1/findings/{codebase_id}`) to persist it.
3. **Invalidate on change.** When files change, call `invalidate_findings`
   (`POST /v1/findings/{codebase_id}/invalidate`) with the changed files so findings whose import-graph
   blast radius is affected are dropped, keeping memory from going stale.

## Rules

- Scope everything by a stable `codebase_id` (e.g. repo slug); the key's org scope isolates tenants.
- No idempotency keys — storing the same finding twice creates two rows unless you dedupe client-side.
- Auth, rate-limit, and `{"detail": …}` error semantics are as in the other ReasonBlocks skills.
