# Scheduling and routing

Scheduling answers two separate questions: **which task is ready?** and **where should its inference run?** Keeping these separate prevents provider or hardware details from leaking into the task graph.

## DAG readiness

A task is `pending`, `ready`, `leased`, `running`, `verifying`, `reviewing`, `accepted`, `blocked`, `failed`, or `cancelled`. It becomes ready when all required dependencies are accepted, its packet revision is valid, required approvals exist, and no conflicting task holds an exclusive resource. Leasing must be atomic and expire if a worker disappears.

Independent ready tasks may run concurrently in separate worktrees. Schema changes can unblock backend and frontend work; integration tests wait for both; final review waits for all required tasks. Cycles are rejected when the graph is created or amended.

## Eligibility before ranking

The router first filters nodes by:

- healthy/reachable status and available session capacity;
- requested model or compatible model class;
- capabilities such as coding, reasoning, or vision;
- context-window and estimated memory requirements;
- provider availability and packet policy.

It then ranks eligible nodes. Version 1 should use explicit, observable heuristics: least active sessions, then configured preference, then stable tie-breaking. Initially prefer one heavy session per machine for predictable memory and latency; operators can raise `maxSessions` after measurement.

## Task-to-model policy

| Work | Default route |
|---|---|
| Mechanical, tightly constrained edits | Small/medium local coder |
| Normal implementation and debugging | Strong local coder |
| Repository investigation | Fast local model |
| Diff and security-oriented review | Local reviewer model |
| Ambiguous architecture/product choice | Frontier model |
| Large context | Model/node with sufficient context and memory |
| Repeated local failure | Stronger local model, then frontier escalation |

V1 policy is configured, not learned. Later scheduling may use historical completion, verification, latency, and retry data, but it must preserve explainable routing and allow operator overrides.

## Retry and reassignment

Failures are classified before retry. A transient endpoint error may retry on another eligible node with the same packet. OOM should lower concurrency or choose a smaller model/more capable node. Repeated semantic or verification failure should not loop forever: preserve evidence, create a review/repair decision, or escalate to a stronger model. Retry budgets belong to attempts and tasks.

Node health uses periodic probes plus request outcomes. Apply timeouts, bounded exponential backoff, jitter, and a cooldown/circuit-breaker state so a sick node is not continuously selected. In-flight work must be idempotently recoverable: an expired lease can be reassigned, while late results from the superseded attempt are retained but cannot win acceptance automatically.

## Follow-ups and conflicts

Worker-proposed tasks enter a review queue. The orchestrator checks whether the work is already represented, attaches dependencies, validates packet quality, and seeks approval where scope or permissions expand. File overlap is only a warning—not proof of conflict—but can guide serialization. Git remains the definitive conflict detector at integration.

