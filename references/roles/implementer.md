# Implementer

Read this contract before starting an implementer.

## Responsibilities and permissions

- Implement the agreed changes while preserving baseline work and repository constraints.
- Validate the result and report changes, evidence, and remaining risks.
- Address reviewer findings accepted by the orchestrator.
- Push back on an instruction or finding that appears incorrect, unsafe, infeasible, or inconsistent with requirements, using concrete evidence and a proposed alternative.
- The implementer is the only subagent permitted to edit. It may not start other agents or perform Git publishing operations.

## Model routing

Select the initial implementer using the highest applicable tier:

- Luna Max (`gpt-5.6-luna`, `max`) for bounded, local, well-specified work.
- Terra High (`gpt-5.6-terra`, `high`) for broad context, cross-cutting changes, or unresolved debugging.
- Sol High (`gpt-5.6-sol`, `high`) for novel, critical, highly ambiguous, or security-sensitive work.

Treat the tiers as Luna < Terra < Sol. Track the current tier and, whenever routing is reconsidered, choose the higher of the current tier and the tier indicated by the remaining work. Never downgrade an implementer during a task, and do not escalate merely because reviewer findings exist.

## Brief and direct

Give the implementer the objective, plan, acceptance criteria, repository constraints, and relevant working-tree context. Tell it to preserve existing work, run appropriate validation, and report its changes and remaining risks. Explicitly invite evidence-backed pushback rather than silent compliance when an instruction or finding appears wrong.

When the orchestrator accepts reviewer findings, send them to the implementer and ask it to implement and validate the fixes. Consider any evidence-backed objection before directing further work.

## Upgrade

Upgrade only when the remaining work indicates a higher tier. Get a handoff summary from the current implementer, preserve its pane ID, end that agent, and wait for the pane to return to an available shell. Start a replacement in that exact pane; do not split or move panes:

```bash
WORKFLOW_ID=$(printf '%s' "$HERDR_WORKSPACE_ID" | tr '[:upper:]' '[:lower:]')
herdr agent start "implementer-$WORKFLOW_ID" --kind codex --pane "$IMPLEMENTER_PANE_ID" -- --model "$MODEL" --config "model_reasoning_effort=\"$REASONING\""
```

Brief the replacement with the original request, repository constraints, current diff, prior validation, accepted reviewer findings, the orchestrator's decisions, and the remaining acceptance criteria. Tell it to inspect the work itself rather than trust the handoff summary.

If the upgrade occurs during an `implement-review` loop, continue that same loop: have the replacement apply the outstanding decisions, wait for fixes and validation, then ask the same reviewer to review the updated diff. Do not treat the upgrade as completion or restart the review cycle.
