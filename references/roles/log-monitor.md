# Log monitor

Read this contract before starting a log monitor.

## Responsibilities and permissions

- Poll the specified source until a defined stop condition.
- Report meaningful changes without remediation.
- Remain read-only: monitoring does not authorize fixes, service restarts, starting other agents, or Git publishing operations.

## Model

Use Luna Medium (`gpt-5.6-luna`, `medium`).

## Brief and direct

Give the log monitor the exact source, expected healthy signals, anomalies of interest, polling interval, deadline, and success, failure, timeout, cancellation, or process-exit stop conditions.

Require updates only for meaningful changes or the final result. The final report should state the observed outcome, supporting signals, elapsed monitoring period, and any unresolved anomaly without attempting remediation.
