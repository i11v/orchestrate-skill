# Orchestrate

An Agent Skill for coordinating and actively steering Codex subagents through Herdr across implementation, review, consultation, and log-monitoring workflows.

Orchestrate delegates execution without delegating responsibility. The primary agent keeps the authoritative task context, selects and briefs specialized subagents, steers their work, resolves disagreements, verifies their claims, and continues until the requested outcome is complete.

## Herdr

The skill uses [Herdr](https://herdr.dev/) to create and manage visible agent panes in the current workspace. Herdr provides the runtime for starting Codex subagents, preserving their sessions, inspecting their state, sending follow-up instructions, and replacing an implementer without losing the surrounding workflow.

Orchestrate expects to run inside an active Herdr workspace with the Herdr skill and CLI available.

## Roles

- **Implementer** — makes the requested changes, validates the result, addresses accepted review findings, and provides evidence-backed pushback when an instruction appears incorrect or unsafe.
- **Reviewer** — independently reviews the task changes without editing and reports actionable findings supported by file and line evidence.
- **Oracle** — gives read-only advice on hard, bounded questions and arbitrates material disagreements that cannot be resolved from requirements, repository evidence, or tests.
- **Log monitor** — watches a specified source until a defined stop condition and reports meaningful changes without performing remediation.

## Workflows

| Mode | Roles | Flow |
| --- | --- | --- |
| `implement-review` | Implementer and reviewer | Implement, review, fix accepted findings, and repeat until no actionable findings remain. |
| `implement-oracle` | Implementer and oracle | Implement while consulting the oracle when a genuinely hard question arises. |
| `implement-only` | Implementer | Implement and validate without an independent review. |
| `log-monitor` | Log monitor | Poll until success, failure, timeout, cancellation, or process exit. |

If no mode is named, Orchestrate infers an unambiguous match and defaults implementation work to `implement-review`.

## Usage

Invoke the skill explicitly and provide the task plus an optional workflow mode:

```text
Use $orchestrate in implement-review mode to implement this change and carry it through independent review.
```

The orchestrator keeps the caller in the current checkout, arranges role panes through Herdr, selects Codex models according to task complexity, and remains responsible for the final result.
