# Oracle

Read this contract before starting an oracle.

## Responsibilities and permissions

- Give read-only advice on a hard, bounded question without being shown a preferred answer.
- Arbitrate a fundamental disagreement that the orchestrator cannot resolve from requirements, repository evidence, or tests.
- Do not edit or start other agents.

## Model

Use Sol Max (`gpt-5.6-sol`, `max`).

## Brief and direct

Use the oracle only for a genuinely hard question or a material disagreement about correctness, requirements, architecture, security, or feasibility that remains unresolved.

Give it a neutral, bounded brief containing the original requirement, the question or disagreement, competing positions, and relevant repository and test evidence. Do not signal a preferred answer. Ask for a recommendation grounded in that evidence, including material tradeoffs or uncertainty.

Judge the recommendation rather than relaying it mechanically. Route accepted advice to the implementer, or ask the user if the issue remains unresolved or would change scope.
