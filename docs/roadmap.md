# Roadmap

Each phase should prove one new dimension without prematurely building the later system.

## Phase 0 — Documentation and architecture (current)

Define boundaries, interfaces, lifecycle, packets, safety, and failure policy. Select a coding-agent runtime only after a thin integration experiment. Manually validate remote Magnitude API behavior.

**Exit:** stakeholders can trace an end-to-end run; major decisions and unknowns are explicit; remote inference is proven manually.

## Phase 1 — Single-machine proof of concept

Build one TypeScript/Node orchestrator process, one repository adapter, one worktree, one local Magnitude endpoint, and one coding-agent process. Accept a manually supplied packet; capture logs, result, diff, and verification.

**Exit:** a bounded task completes reproducibly without manual edits, and failure leaves inspectable state.

## Phase 2 — Frontier planning

Let a frontier model inspect the repository, generate validated structured packets, and review one local worker result. Add clarification and escalation boundaries.

**Exit:** one user goal becomes an auditable packet whose result survives frontier review.

## Phase 3 — Multi-worker local execution

Add multiple worktrees, task queue, DAG readiness, parallel agent processes, reviewer roles, and controlled follow-up proposals.

**Exit:** independent tasks execute concurrently and dependent tasks unblock deterministically.

## Phase 4 — Multi-node inference cluster

Add node registry, health probes, multiple Magnitude endpoints, capability/model matching, capacity limits, retries, and least-loaded routing.

**Exit:** tasks survive loss of one inference node and routing decisions are explainable.

## Phase 5 — Robust orchestration

Make SQLite state authoritative across restarts. Add leases, resumable runs, attempt history, recovery reconciliation, escalation policies, and human approval gates.

**Exit:** a killed orchestrator resumes without duplicate authoritative work or lost evidence.

## Phase 6 — Dashboard

Expose live tasks, nodes, models, logs, verification, approvals, DAG state, usage, and estimated avoided API cost through a web dashboard. Use SSE or WebSocket only when live updates justify it.

**Exit:** an operator can understand and control a run without reading raw database state.

## Phase 7 — Advanced scheduling

Use model performance history for task/model matching, reviewer assignment, concurrency tuning, and adaptive escalation while retaining explicit policy and overrides.

**Exit:** historical routing improves measured outcomes without obscuring why a route was chosen.

## Phase 8 — Optional research

Explore distributed/sharded inference, Thunderbolt/RDMA experiments, larger shared models, or speculative multi-model workflows. These are experiments, not dependencies of the core product.

## V1 non-goals

- Custom model runtime, Metal kernels, or distributed inference protocol
- Swift requirement, Kubernetes, or distributed shared memory
- Cross-machine filesystem synchronization or custom git implementation
- Model training or fine-tuning
- Multi-machine model sharding
- Automatic production deployment or autonomous merging by default
- Complex UI before the execution path is reliable

