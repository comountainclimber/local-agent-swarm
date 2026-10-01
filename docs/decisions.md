# Architectural decisions

This is a living decision log. “Accepted” decisions define the current plan; changes should add a superseding entry and rationale rather than silently rewriting history.

## ADR-001 — One control plane owns execution

**Status:** Accepted for v1

The repository, worktrees, coding-agent processes, tools, services, state, and integration live on one control-plane machine. Remote hosts may be inference-only.

**Why:** It avoids source synchronization, distributed tool execution, and split-brain git state while still using all inference hardware. The control plane is initially a single point of failure, mitigated by durable state and recoverable git artifacts.

## ADR-002 — HTTP endpoint abstraction

**Status:** Accepted

Inference nodes expose ordinary compatible HTTP APIs. The orchestrator stores endpoint URLs and is transport-independent.

**Why:** Wi-Fi, Ethernet, Thunderbolt Bridge, and private overlays already provide IP networking. A custom protocol, RDMA, or shared-memory abstraction would add risk without helping the initial independent-node workload.

## ADR-003 — Provider-neutral inference contract

**Status:** Accepted

Workers depend on a normalized streaming `InferenceProvider`, with Magnitude as the first adapter rather than a system-wide dependency.

**Why:** Model serving and orchestration evolve independently. This also enables frontier and generic compatible providers without branching worker logic.

## ADR-004 — Git worktree per task

**Status:** Accepted

Each executing task gets a dedicated branch, worktree, process, logs, and result.

**Why:** Worktrees give inexpensive source isolation using standard git, support parallelism, and leave inspectable diffs. They do not provide OS security isolation, which remains a separate concern.

## ADR-005 — DAG, packets, and evidence are first-class

**Status:** Accepted

Features decompose into dependency graphs of versioned work packets. Completion requires structured results and verification evidence.

**Why:** Parallelism depends on explicit dependencies; reliable local-model work depends on narrow contracts; recovery depends on persistent evidence.

## ADR-006 — SQLite before distributed state

**Status:** Accepted for initial implementation

Use SQLite for run, task, node, attempt, verification, artifact, and event state.

**Why:** One control-plane writer does not justify a distributed database. SQLite supports transactional leases and restart recovery with minimal operations. Revisit only when measured concurrency or multi-control-plane requirements demand it.

## ADR-007 — TypeScript/Node for the orchestrator

**Status:** Preferred; confirm during Phase 1

Implement the orchestrator in TypeScript/Node. A later dashboard can share types and ecosystem tooling. SwiftUI may be considered for an optional native client, not the core.

## ADR-008 — Explicit heuristics before adaptive routing

**Status:** Accepted

V1 filters by health/capability/model/capacity and ranks by simple configured preferences and load.

**Why:** Explainable policy is easier to test and debug. Learning from history comes after trustworthy outcome data exists.

## ADR-009 — Human-controlled high-impact actions

**Status:** Accepted

Merging autonomously is off by default. Push, production, destructive database, deployment, and secrets-sensitive actions require configured approval.

**Why:** Model review complements but does not replace authorization. Permissions must be enforced where tools execute.

## ADR-010 — No sharding in v1

**Status:** Accepted

Multiple hosts serve independent inference sessions. Distributed model sharding is optional future research.

**Why:** Parallel independent brains provide immediate throughput without coupling the first implementation to hard distributed-systems and hardware problems.

## Open decisions for implementation discovery

- Which coding-agent runtime has the best noninteractive protocol, cancellation, event capture, and provider override behavior?
- What exact normalized tool-call and usage semantics cover the tested Magnitude APIs?
- Is one branch per task sufficient for repairs, or should repairs use child branches by default?
- Which sandbox mechanism is practical for arbitrary shell permissions on macOS?
- What artifact retention and redaction defaults balance debugging, privacy, and disk use?

