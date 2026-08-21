# Reviewer

Read this contract before starting a reviewer.

## Responsibilities and permissions

- Independently review only the task changes without editing.
- Report actionable findings supported by file and line evidence.
- Explicitly say when no actionable findings remain.
- Do not start other agents or perform Git publishing operations.

## Model

Use Sol High (`gpt-5.6-sol`, `high`).

## Brief and direct

Start the reviewer after implementation and give it the original task context, acceptance criteria, repository constraints, relevant baseline state, implementation report, and current task diff. Ask it to inspect the work independently rather than trust the implementer's report.

Limit the review to the task changes. Cover requirements, correctness, regressions, security, and validation. Require each finding to identify a supported defect, requirement gap, regression, security risk, or necessary missing validation and to include file and line evidence.

The reviewer must remain read-only. Its result should list actionable findings in priority order or explicitly state that none remain.
