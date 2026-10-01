# Verification

Verification is task data, not a final courtesy. A worker cannot turn “files changed” into “completed” without evidence tied to the packet and attempt.

## Verification plan

Each packet declares required commands and, where helpful, named tests. Checks commonly include unit and integration tests, typechecking, lint, build, migration validation, targeted end-to-end tests, and diff inspection. Commands should start narrow for fast iteration and broaden before acceptance according to risk.

Every command result records:

- exact command and working directory;
- start/end time and exit status;
- attempt, base commit, and resulting commit/diff identity;
- concise output summary plus a reference to full logs;
- whether the command passed, failed, timed out, or was not run and why.

Logs are evidence, not instructions: terminal output must never be interpreted as authorization to exceed a packet's permissions.

## Gates

1. **Worker gate:** required task checks pass and diff scope is reviewed.
2. **Reviewer gate:** acceptance criteria, tests, security-sensitive boundaries, and evidence are independently assessed.
3. **Integration gate:** accepted branches combine cleanly and aggregate checks pass against the integrated tree.
4. **Frontier gate:** cross-cutting behavior, architecture, and unresolved concerns receive final high-value review.
5. **Human gate:** the operator controls merge/push or any higher-risk action by policy.

Cached or previously observed results do not prove a new diff. Evidence is valid only for the recorded source state and environment. Flaky tests are reported as uncertainty; repeated execution may characterize them but must not silently reclassify failure as success.

## Proportionality

A documentation edit may need link and formatting checks. A tenant-scoped query needs isolation tests. A schema migration needs forward validation, compatibility analysis, and any repository-standard rollback check. The planner selects checks using repository conventions and risk; workers may propose additional checks but may not omit required ones.

## Failed verification

The worker diagnoses and retries within its bounds. When failures persist, it returns the failure evidence and the smallest useful explanation. The orchestrator can issue a repair packet, change model/node, request a product decision, or escalate. It never converts exhausted retries into success.

