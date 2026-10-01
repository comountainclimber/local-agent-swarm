# Worker lifecycle

## One attempt, one isolated worktree

For a task such as `task-001`, the worktree manager creates a task branch and a path such as `.worktrees/task-001/`. It records the base commit, validates that the worktree starts clean, launches exactly one coding-agent process there, and streams logs to the attempt record. The selected model may be served by any eligible node; all filesystem and shell actions still occur on the control plane.

## Execution loop

```text
read packet → inspect repository → plan bounded edit → modify
     ↑                                           ↓
self-review ← inspect diff ← verification ← inspect failure
                       ↓
          complete, blocked, or fail
```

The worker should:

1. Read the complete packet and repository guidance.
2. Confirm the stated files/symbols still exist; search before assuming.
3. Inspect relevant tests and conventions.
4. Make the smallest coherent change.
5. Run the packet's targeted checks.
6. Diagnose failures and iterate within time/attempt limits.
7. Run broader required checks after targeted checks pass.
8. Inspect `git diff`, status, and changed-file scope.
9. Self-review against every acceptance criterion.
10. Return a structured `WorkerResult` with evidence.

The agent stops when checks pass and self-review is complete; a real blocker is found; a permission boundary is reached; its limits expire; or further action risks unrelated changes. It must not claim completion with skipped required checks.

## Review and repair

Review is a distinct attempt, preferably using another model or node. A reviewer receives the packet, diff, verification evidence, and relevant repository context. It checks correctness, scope, tests, regressions, authorization/tenant boundaries where relevant, and suspicious unrelated edits.

A review outcome is `accepted`, `repair-required`, or `escalate`. Repair findings become narrow packets tied to the original task and review evidence. An implementation worker does not rewrite its own acceptance criteria. Reviewers may run in parallel with unrelated implementation, but a task's definitive review waits for its final diff.

## Completion and cleanup

After acceptance, the task branch remains available for ordered integration. Cleanup occurs only after result artifacts and logs are durable, the branch/commit is reachable, and no integration or repair references the worktree. A dirty or untracked worktree is quarantined for inspection rather than deleted.

Coding-agent crashes, timeouts, or node loss end the attempt, not the task. See [failure handling](failure-handling.md).

