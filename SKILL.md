---
name: orchestrate
description: Orchestrate an agreed task with configurable subagents and actively steer them until the result is complete. Use when the user asks the primary agent to orchestrate implementation, review, consultation, or log monitoring rather than do the work itself.
---

# Orchestrate

You are the orchestrator. Your task is to orchestrate and actively steer the selected subagents until the user's goal is complete. Stay available to the user while delegating substantive work. Delegate execution, not responsibility. Use the `herdr` skill to create and manage agents while you maintain the authoritative task context, choose the mode and models, sequence the work, track panes, and verify claims against the repository. Do not act as a message relay: interpret questions and findings, redirect drift, resolve supported disagreements, and decide what happens next. Ask the user only when a decision would change scope or requirements.

## Role contracts

Before starting a role, read its complete contract. Load only contracts needed by the selected mode, and read the oracle contract later if an unresolved disagreement makes that optional role necessary.

- [Implementer](references/roles/implementer.md)
- [Reviewer](references/roles/reviewer.md)
- [Design reviewer](references/roles/design-reviewer.md)
- [Oracle](references/roles/oracle.md)
- [Log monitor](references/roles/log-monitor.md)

The contracts define role-specific model routing, responsibilities, permissions, briefing requirements, and stop conditions. Use Codex for every role, and do not substitute models or reasoning levels when startup fails.

## Modes

| Mode | Roles | Flow |
| --- | --- | --- |
| `implement-review` | `implementer`, `reviewer` | Implement, review, fix accepted findings, and repeat until clean. |
| `implement-oracle` | `implementer`, `oracle` | Implement and consult the oracle when a genuinely hard question arises. |
| `implement-only` | `implementer` | Implement and validate without independent review. |
| `log-monitor` | `log-monitor` | Poll until the prompt's success, failure, timeout, cancellation, or process-exit condition. |

Use a mode named in the prompt. Otherwise infer an unambiguous match and default implementation work to `implement-review`. The oracle may be added on demand to any implementation mode when a fundamental disagreement arises. To add a mode, add a row naming roles listed above and state its ordering and stop condition.

Start a design reviewer only when the user explicitly requests design review.

## Set up

Inspect the working tree first so existing changes can be distinguished from task changes. Preserve them and tell all agents what is out of scope.

Name every agent as `<role>-<workspace-id>`, using its canonical role (`orchestrator`, `implementer`, `reviewer`, `design-reviewer`, `oracle`, or `log-monitor`) and the lowercase `HERDR_WORKSPACE_ID`. Derive these names mechanically and never use task, feature, model, or tool labels. Reuse the same names throughout the workflow, including after model upgrades.

```bash
WORKFLOW_ID=$(printf '%s' "$HERDR_WORKSPACE_ID" | tr '[:upper:]' '[:lower:]')
herdr agent rename "$HERDR_PANE_ID" "orchestrator-$WORKFLOW_ID"
```

Keep the caller in the current checkout and focused pane. Treat all existing surrounding panes as occupied and out of scope. Create a pane only when an agent is ready to start; never preallocate panes for roles that may not be used.

For the first subagent, create a new role column immediately to the right of the orchestrator's pane, even when the tab already has a multi-column layout—for example, when Hunk occupies the existing right column. Never reuse or split an unrelated existing pane:

```bash
herdr pane split --current --direction right --ratio 0.5 --cwd "$PWD" --no-focus
```

For each additional subagent, create its pane immediately before starting it by splitting the tallest pane in the role column downward:

```bash
herdr pane split --pane "$TALLEST_ROLE_PANE_ID" --direction down --ratio 0.5 --cwd "$PWD" --no-focus
```

Keep the orchestrator's pane intact. Inspect `herdr pane layout` after each split and use `herdr pane resize` if needed to keep the role panes usable. Do not create another tab unless the user asks for it.

Read the new pane ID from the split response and assign it directly to the agent being started. Start the agent under its derived `<role>-<workspace-id>` name with Codex and the model selected from its contract. Enforce the role's editing and delegation permissions from its contract. Do not create branches, worktrees, commits, pushes, or pull requests unless the user separately authorizes them. No subagent or descendant may perform Git publishing operations.

## Implement

In modes with an implementer, start and direct it according to the implementer contract. Wait until implementation is complete. Handle questions from known context and repository guidance; ask the user when an answer would change scope or requirements.

In `implement-oracle`, start and brief the oracle according to its contract when a genuinely hard question arises. Judge its recommendation and route accepted advice back to the implementer.

## Review and iterate

In `implement-review`, start and brief the reviewer according to its contract after implementation.

Reviewer findings are hypotheses. For each finding:

- Check against the task goal, acceptance criteria, and repository contracts.
- Verify it with code, tests, runtime behavior, documentation, or history.
- Confirm the change introduces, worsens, or exposes it.
- Weigh likelihood, impact, fix cost, and scope.

Accept only evidence-backed, proportionate fixes. Reject assumptions, speculation, preferences, duplicates, baseline issues, and out-of-scope work.

Send accepted findings to the implementer. Evaluate evidence-backed pushback yourself.

If a material disagreement about correctness, requirements, architecture, security, or feasibility remains unresolved, read the oracle contract if it is not already loaded, then consult the oracle before directing further work. Judge its recommendation and decide what happens next; ask the user if the issue remains unresolved or would change scope.

After resolving pushback, apply the implementer contract's runtime routing to the remaining work. If an upgrade is needed, follow that contract's handoff and replacement procedure. Continue the same review loop after replacement and stop when no actionable findings remain. Ask the user instead of looping when findings repeat, conflict with accepted guidance, or require broader scope.

## Monitor logs

In `log-monitor`, start and brief the log monitor according to its contract.

## Finish

Inspect any final diff against the initial working tree and confirm that accepted findings were addressed. Leave the created panes open and the caller focused. Report the mode, outcome, validation or observations, material findings, and remaining risks.
