---
name: Track a monitored agent run with telemetry
description: Start a monitor run, log per-step telemetry, evaluate the six server-side trajectory monitors mid-run for steering text, and finish the run — the ReasonBlocks monitor telemetry lifecycle.
api: openapi/reasonblocks-openapi-original.json
operations:
- start_run_v1_monitor_runs_post
- log_step_v1_monitor_runs__run_id__steps_post
- evaluate_monitors_v1_monitors_evaluate_post
- finish_run_v1_monitor_runs__run_id__finish_post
---

# Track a monitored agent run with telemetry

Instrument an agent run so ReasonBlocks can detect failure patterns (loops, redundant work) mid-run and
return steering text to append to the system message. Base URL `https://rb-api.reasonblocks.com`, bearer auth.

## Steps

1. **Start the run.** Call `start_run` (`POST /v1/monitor/runs`) with run metadata (and optionally a task
   profile: `coding`, `pr_review`, or `qa`). Keep the returned `run_id`.
2. **Log each step.** Call `log_step` (`POST /v1/monitor/runs/{run_id}/steps`) with the step payload; the
   server returns the scored bundle (FSM state + any steering).
3. **Evaluate monitors (optional, mid-run).** Call `evaluate_monitors` (`POST /v1/monitors/evaluate`) to run
   the six trajectory monitors and get steering text to inject when the agent is looping or stuck.
4. **Finish the run.** Call `finish_run` (`POST /v1/monitor/runs/{run_id}/finish`) to close the run so it
   settles in the dashboard and health summary.

## Rules

- Always `finish_run` — unfinished runs show as stuck dashboard rows.
- Monitor weights follow the built-in task profile unless overridden per call.
- Auth, rate-limit (200/sec, `429` + `Retry-After`), and `{"detail": …}` error rules are as in the
  retrieve-and-steer skill. Live updates stream from `GET /v1/monitor/stream` (SSE) when the server has
  `SUPABASE_DB_URL` configured.
