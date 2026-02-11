# Ralph-RLM Getting Started Guide

This guide gets you from zero to a working Ralph + RLM loop quickly.

## 1) Install and verify plugin load

- Put the plugin at `.opencode/plugins/ralph-rlm.ts` (project) or `~/.config/opencode/plugins/` (global).
- Start OpenCode in your repo.
- Ensure `.opencode/ralph.json` exists (or run doctor in step 2).

## 1.5) Understand the roles (supervisor vs Ralph vs worker)

Ralph-RLM has three distinct layers so the agent behavior stays predictable:

- **Supervisor (your current session):** Orchestrates the loop and responds to `ralph_ask()` questions. It should not implement code. It monitors progress and controls lifecycle (pause/resume/end).
- **Ralph strategist (per-attempt):** A short-lived planning session created by the plugin. It reviews outcomes, updates `PLAN.md` / `RLM_INSTRUCTIONS.md`, then calls `ralph_spawn_worker()`.
- **RLM worker (per-attempt):** A fresh coding session that does the actual edits, calls `ralph_verify()`, then stops.

Think of it as: **Supervisor** (you) → **Ralph strategist** (planning) → **RLM worker** (implementation).

### Quick mental model

```
Supervisor (you)
  └─ Ralph strategist (attempt N)
       └─ RLM worker (attempt N)
            └─ ralph_verify()
                ├─ pass → done
                └─ fail → attempt N+1 (new Ralph + worker)
```

### Expected behavior checklist

- Supervisor stays high-level (orchestration, decisions, answering `ralph_ask`).
- Ralph strategist updates `PLAN.md` / `RLM_INSTRUCTIONS.md`, then spawns a worker.
- Worker does the actual edits, calls `ralph_verify()`, then stops.
- The loop continues until `verify.command` passes or `maxAttempts` is reached.

## 2) Bootstrap setup

Run:

`ralph_doctor(autofix=true)`

This creates missing baseline files/config and checks readiness.

## 3) Create a real plan

Fast path (recommended):

`ralph_quickstart_wizard(goal, requirements, stopping_conditions, features, steps, todos, start_loop=false)`

Manual path:

- `ralph_bootstrap_plan(...)`
- `ralph_validate_plan()`

Tip: put clear stopping conditions in the plan and a strong `verify.command` in `.opencode/ralph.json`.

## 4) Start supervision

Default is manual start (`autoStartOnMainIdle: false`):

`ralph_create_supervisor_session(start_loop=true)`

Check status anytime:

`ralph_supervision_status()`

## 4.5) Use the included agent profiles (recommended)

This repo includes project-local OpenCode agent profiles in `.opencode/agents/`:

- `supervisor` (primary): safe orchestration posture for Ralph loop control
- `ralph-reviewer` (subagent): read-only quality review
- `docs-writer` (subagent): documentation edits without shell access
- `security-auditor` (subagent): read-only security-focused review

Use these to control behavior and delegation while keeping loop execution in the plugin.

## 5) Monitor progress

- Structured feed: `SUPERVISOR_LOG.md`
- Human-readable timeline: `CONVERSATION.md`
- Session questions: use `ralph_respond(id, answer)` when prompted by `ralph_ask()`
- Quick peek (posts worker CURRENT_STATE into main conversation): `ralph_peek_worker()`

### TUI navigation tips (seeing child sessions)

- Cycle primary agents with Tab (OpenCode calls this `agent_cycle`).
- If sub-agents spawn child sessions, cycle parent ⇄ child with:
  - `<leader>+Right` (`session_child_cycle`)
  - `<leader>+Left` (`session_child_cycle_reverse`)
- Open the sessions list with `/sessions` (or `<leader>+l`) to jump between sessions.
- Keybinds are configured in `opencode.json` (default leader is `ctrl+x`).

## 6) Control loop lifecycle

- Pause without ending: `ralph_pause_supervision(reason?)`
- Resume: `ralph_resume_supervision(start_loop?)`
- Hard stop: `ralph_end_supervision(reason?, clear_binding?, delete_sessions?)`
- Restart after stop/done: `ralph_create_supervisor_session(restart_if_done=true)`

## 7) Optional reviewer flow (gated)

To avoid running reviewer too often:

1. Worker signals readiness: `ralph_request_review("ready for review")`
2. Supervisor runs reviewer: `ralph_run_reviewer()`
3. Report is written to `.opencode/reviews/review-attempt-N.md`

Reviewer limits are controlled by:

- `reviewerRequireExplicitReady`
- `reviewerMaxRunsPerAttempt`
- `reviewerEnabled`

Reviewer runtime state persists in `.opencode/reviewer_state.json`.

## 8) Recommended baseline config

```json
{
  "enabled": true,
  "autoStartOnMainIdle": false,
  "statusVerbosity": "normal",
  "maxAttempts": 25,
  "heartbeatMinutes": 15,
  "verifyTimeoutMinutes": 15,
  "verify": { "command": ["bun", "run", "verify"], "cwd": "." },
  "gateDestructiveToolsUntilContextLoaded": true,
  "maxRlmSliceLines": 200,
  "requireGrepBeforeLargeSlice": true,
  "grepRequiredThresholdLines": 120,
  "subAgentEnabled": true,
  "maxSubAgents": 5,
  "maxConversationLines": 1200,
  "conversationArchiveCount": 3,
  "reviewerEnabled": false,
  "reviewerRequireExplicitReady": true,
  "reviewerMaxRunsPerAttempt": 1,
  "reviewerOutputDir": ".opencode/reviews",
  "reviewerPostToConversation": true,
  "agentMdPath": "AGENT.md"
}
```

## 9) Troubleshooting

- `ralph_spawn_worker() can only be called from a Ralph strategist session`
  - Expected in main session. Start with `ralph_create_supervisor_session()` and let strategist spawn workers.
- Loop not starting
  - Check `ralph_supervision_status()` and run `ralph_doctor(autofix=true)`.
- Verify always failing
  - Fix `verify.command` to match your repo’s actual quality gate.
- Too much log noise
  - Lower `statusVerbosity` or reduce loop chatter by using milestone-level `ralph_report` only.

---

For full reference, see `README.md`.
