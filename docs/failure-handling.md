# Failure handling and recovery

Failures end attempts; they should not corrupt run state or erase evidence. Persist task intent, packet revision, base commit, leases, events, logs, verification, and produced commits so the orchestrator can resume after a process or machine restart.

## Failure policy

| Failure | Detection | Initial response | Escalation |
|---|---|---|---|
| Node offline/asleep | Failed probes and requests | Stop routing; expire/reassign leases | Operator if no eligible node |
| Endpoint timeout/stall | Layered deadlines | Cancel; retry with backoff, possibly elsewhere | Stronger/different provider after budget |
| Model crash/OOM | Normalized provider error | Reduce concurrency or change model/node | Operator for repeated capacity mismatch |
| Coding-agent crash | Process exit/heartbeat loss | Preserve worktree/logs; create new attempt | Inspect repeated deterministic crash |
| Verification failure | Nonzero check/result | Iterate within limits | Repair task or frontier review |
| Worker loop too long | Time/step/token budget | Graceful cancel and checkpoint evidence | Re-scope or stronger model |
| Unrelated changes | Changed-file/diff review | Reject or request focused repair | Human if intent is ambiguous |
| Dirty worktree | Pre/postflight status | Quarantine; never delete automatically | Operator resolves ownership |
| Merge conflict | Git integration | Keep task commits; stop integration | Repair/rebase packet with dependency context |
| Task blocked | Structured result | Record blocker; release capacity | Planner/human resolves dependency or decision |
| Frontier API failure | Provider error | Bounded retry/fallback if configured | Pause at decision boundary |

## Recovery invariants

- A task has at most one authoritative active lease.
- Attempts and their events are immutable after closure.
- Retrying never overwrites the previous worktree evidence.
- Late results from expired leases cannot be accepted without reconciliation.
- A branch/commit remains reachable before worktree removal.
- Destructive cleanup never targets an unresolved or dirty worktree.
- External side effects use idempotency keys or explicit human confirmation where available.

## Resume procedure

On startup, the orchestrator loads durable state, marks stale leases suspect, reconciles live processes and worktrees, probes nodes, and appends recovery events. It does not assume `running` means a process exists. Safe completed results return to review/integration; interrupted attempts are closed and reissued only after their filesystem state is classified.

Retry decisions use failure class, not just a counter. Infrastructure failures usually retain the same packet. Semantic failures may need better context, a repair packet, or frontier escalation. Permission/product blockers require a decision, not more tokens.

