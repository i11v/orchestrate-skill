---
name: orchestrate
description: Orchestrate an agreed task with configurable subagents and actively steer them until the result is complete. Use when the user asks the primary agent to orchestrate implementation, review, consultation, or log monitoring rather than do the work itself.
---

# Orchestrate

You are the orchestrator. Your task is to orchestrate and actively steer the selected subagents until the user's goal is complete. Delegate execution, not responsibility. Use the `herdr` skill to create and manage agents while you maintain the authoritative task context, choose the mode and models, sequence the work, track panes, and verify claims against the repository. Do not act as a message relay: interpret questions and findings, redirect drift, resolve supported disagreements, and decide what happens next. Ask the user only when a decision would change scope or requirements.

## Subagent roles

### Implementer

Responsibilities:

- Implement the agreed changes while preserving baseline work and repository constraints.
- Validate the result and report changes, evidence, and remaining risks.
- Address reviewer findings accepted by the orchestrator.
- Push back on an instruction or finding that appears incorrect, unsafe, infeasible, or inconsistent with requirements, using concrete evidence and a proposed alternative.

Models:

- Luna Max (`gpt-5.6-luna`, `max`) for bounded, local, well-specified work.
- Terra High (`gpt-5.6-terra`, `high`) for broad context, cross-cutting changes, or unresolved debugging.
- Sol High (`gpt-5.6-sol`, `high`) for novel, critical, highly ambiguous, or security-sensitive work.

### Reviewer

Responsibilities:

- Independently review only the task changes without editing.
- Report actionable, evidence-backed findings and explicitly say when none remain.

Models:

- Sol High (`gpt-5.6-sol`, `high`).

### Oracle

Responsibilities:

- Give read-only advice on a hard, bounded question without being shown a preferred answer.
- Arbitrate a fundamental disagreement that the orchestrator cannot resolve from requirements, repository evidence, or tests.

Models:

- Sol Max (`gpt-5.6-sol`, `max`).

### Log monitor

Responsibilities:

- Poll the specified source until a defined stop condition.
- Report meaningful changes without remediation.

Models:

- Luna Medium (`gpt-5.6-luna`, `medium`).

## Runtime routing

Use Codex for every role. Select the initial implementer using the highest applicable model tier listed under its role. Treat the implementation tiers as Luna < Terra < Sol. Track the current tier and, whenever routing is reconsidered, choose the higher of the current tier and the tier indicated by the remaining work. Never downgrade an implementer during a task. Do not substitute models or reasoning levels when startup fails.

## Modes

| Mode | Roles | Flow |
| --- | --- | --- |
| `implement-review` | `implementer`, `reviewer` | Implement, review, fix accepted findings, and repeat until clean. |
| `implement-oracle` | `implementer`, `oracle` | Implement and consult the oracle when a genuinely hard question arises. |
| `implement-only` | `implementer` | Implement and validate without independent review. |
| `log-monitor` | `log-monitor` | Poll until the prompt's success, failure, timeout, cancellation, or process-exit condition. |

Use a mode named in the prompt. Otherwise infer an unambiguous match and default implementation work to `implement-review`. The oracle may be added on demand to any implementation mode when a fundamental disagreement arises. To add a mode, add a row naming roles listed above and state its ordering and stop condition.

## Set up

Inspect the working tree first so existing changes can be distinguished from task changes. Preserve them and tell all agents what is out of scope.

Name every agent as `<role>-<workspace-id>`, using its canonical role (`orchestrator`, `implementer`, `reviewer`, `oracle`, or `log-monitor`) and the lowercase `HERDR_WORKSPACE_ID`. Derive these names mechanically and never use task, feature, model, or tool labels. Reuse the same names throughout the workflow, including after model upgrades.

```bash
WORKFLOW_ID=$(printf '%s' "$HERDR_WORKSPACE_ID" | tr '[:upper:]' '[:lower:]')
herdr agent rename "$HERDR_PANE_ID" "orchestrator-$WORKFLOW_ID"
```

Keep the caller in the current checkout and focused pane. Let `ROLE_COUNT` be the number of roles in the selected mode. Create one right-hand role pane first:

```bash
herdr pane split --current --direction right --ratio 0.5 --cwd "$PWD" --no-focus
```

If `ROLE_COUNT` is `1`, stop splitting. If it is `2`, split the first role pane so both agents share the right column:

```bash
herdr pane split --pane "$FIRST_ROLE_PANE_ID" --direction down --ratio 0.5 --cwd "$PWD" --no-focus
```

For larger future modes, keep the orchestrator's left half intact and repeatedly split the tallest role pane downward until there is one pane per role. If an oracle is needed later and has no pane, add one using the same rule and start the oracle there. Inspect `herdr pane layout` after each split and use `herdr pane resize` if needed to keep the role panes usable. Do not create another tab unless the user asks for it.

Read pane IDs from the responses and map them to roles in mode order. Start each agent under its derived `<role>-<workspace-id>` name with Codex and the model selected by runtime routing. Do not create branches, worktrees, commits, pushes, or pull requests unless the user separately authorizes them. Only the implementer may edit; no subagent may start other agents or perform Git publishing operations.

## Implement

In modes with an implementer, instruct it to carry out the agreed task. Give it the objective, plan, acceptance criteria, repository constraints, and relevant working-tree context. Ask it to preserve existing work, run appropriate validation, and report its changes and remaining risks. Explicitly invite evidence-backed pushback rather than silent compliance when an instruction or finding appears wrong.

Wait until implementation is complete. Handle questions from known context and repository guidance; ask the user when an answer would change scope or requirements.

In `implement-oracle`, send genuinely hard questions to the oracle with the relevant evidence but without a preferred answer. Judge its recommendation and route accepted advice back to the implementer.

## Review and iterate

In `implement-review`, start the reviewer after implementation and give it the same task context. Ask for an independent review of only the task changes, with actionable findings supported by file and line evidence. The review should cover requirements, correctness, regressions, security, and validation, and explicitly say when no actionable findings remain.

Judge the findings yourself rather than relaying them mechanically. Accept findings that identify a supported defect, requirement gap, regression or security risk, or necessary missing validation. Reject baseline issues, unsupported speculation, duplicates, subjective preferences, and out-of-scope enhancements.

Send accepted findings to the implementer and consider any evidence-backed pushback. Recheck the relevant requirements, code, and tests, and revise your decision when the objection is supported.

If a material disagreement about correctness, requirements, architecture, security, or feasibility remains unresolved, consult the oracle before directing further work. Give it a neutral, bounded brief containing the original requirement, the finding or instruction, the implementer's objection, and the relevant evidence without signaling a preferred answer. Judge its recommendation and decide what happens next; ask the user if the issue remains unresolved or would change scope.

After resolving pushback, apply runtime routing again to the remaining work. Decide whether the implementer should stay at its current tier or move higher because the findings reveal greater complexity or risk; do not escalate merely because findings exist.

When upgrading, get a handoff summary from the current implementer, preserve its pane ID, end that agent, and wait for the pane to return to an available shell. Start a new Codex agent in that exact pane; do not split or move panes:

```bash
WORKFLOW_ID=$(printf '%s' "$HERDR_WORKSPACE_ID" | tr '[:upper:]' '[:lower:]')
herdr agent start "implementer-$WORKFLOW_ID" --kind codex --pane "$IMPLEMENTER_PANE_ID" -- --model "$MODEL" --config "model_reasoning_effort=\"$REASONING\""
```

Brief the replacement with the original request, repository constraints, current diff, prior validation, accepted reviewer findings, the orchestrator's decisions, and the remaining acceptance criteria. Tell it to inspect the work itself rather than trust the handoff summary.

If the upgrade occurs during an `implement-review` loop, continue that same loop: have the replacement implementer apply the outstanding decisions, wait for fixes and validation, then ask the same reviewer to review the updated diff. Do not treat the upgrade as completion or restart the review cycle. Stop when no actionable findings remain. Ask the user instead of looping when findings repeat, conflict with accepted guidance, or require broader scope.

## Monitor logs

In `log-monitor`, give the agent the exact source, expected healthy signals, anomalies of interest, polling interval, deadline, and stop conditions. Require updates only for meaningful changes or the final result. Monitoring does not authorize fixes or service restarts.

## Finish

Inspect any final diff against the initial working tree and confirm that accepted findings were addressed. Leave the created panes open and the caller focused. Report the mode, outcome, validation or observations, material findings, and remaining risks.
