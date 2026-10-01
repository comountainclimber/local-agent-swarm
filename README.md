# Local Agent Swarm

Local Agent Swarm is a design for a local-first, multi-agent software-engineering orchestrator. A frontier model acts like a staff engineer: it resolves ambiguity, studies the repository, creates a dependency-aware plan, and reviews the integrated result. Local open-weight models do the iteration-heavy work—implementation, tests, migrations, debugging, and review—at effectively zero marginal token cost.

This repository is the Phase 0 design, not an implementation. It deliberately contains no runtime, application code, or package manifest.

## Core architecture

One **control-plane machine** owns the repository, git worktrees, agent processes, tools, services, durable state, and integration. Any number of **inference nodes** expose ordinary HTTP endpoints. A coding agent's hands operate in a local worktree while its model's brain may run on another Mac or GPU host.

```text
user → frontier planner → task DAG → scheduler → coding agents in worktrees
                                            ↘ HTTP inference nodes
                            verification → review → human-approved integration
```

The scheduler knows endpoint URLs, capabilities, models, health, and capacity. It does not know whether traffic uses Wi-Fi, Ethernet, Thunderbolt Bridge, or a private overlay network. Magnitude is the preferred first backend, behind a replaceable provider interface.

## Why a hybrid system

Frontier intelligence is best spent on unclear requirements, architecture, decomposition, escalation, and final judgment. Local models are best used where a tight work packet and executable checks can turn abundant inference into reliable iteration. Explicit scope, acceptance criteria, and verification make smaller models far more effective than a single broad prompt.

## Example four-Mac deployment

- **Mac 1 — control plane:** repository, worktrees, orchestrator, agents, tests, Docker, frontier calls, and optionally local inference.
- **Macs 2–4 — inference nodes:** Magnitude serving coding or reviewer models over HTTP.

Four machines are only an example. The same registry can later include more Macs, Linux GPU boxes, or rented workers. Version 1 uses independent inference nodes—not cross-machine model sharding.

## Design principles

1. Frontier intelligence is scarce; local tokens are cheap.
2. Plan once, execute many.
3. Tight scopes beat giant prompts.
4. Every task must be verifiable.
5. Worktrees are the unit of isolation.
6. Inference location is independent from execution location.
7. Providers are replaceable; hardware nodes are disposable.
8. Failures create durable evidence, not chaos.
9. Humans retain final control.
10. Prefer simple protocols and the smallest useful orchestrator.

## Status and roadmap

**Current status: Phase 0 — architecture and planning.** The proposed implementation starts with one manually supplied packet, one worktree, and one Magnitude endpoint. Frontier planning, parallel workers, multi-node routing, durable recovery, a dashboard, and adaptive scheduling follow only after that path is proven.

See the [roadmap](docs/roadmap.md) for phases and exit criteria.

## Documentation

- [Architecture](docs/architecture.md) — boundaries, components, state, and provider contracts
- [Work packets](docs/work-packets.md) — task contracts and structured results
- [Worker lifecycle](docs/worker-lifecycle.md) — iterative execution and cleanup
- [Scheduling](docs/scheduling.md) — DAG readiness, routing, and escalation
- [Networking](docs/networking.md) — endpoint-based multi-host connectivity
- [Magnitude](docs/magnitude.md) — preferred, replaceable local inference backend
- [Verification](docs/verification.md) — evidence and integration gates
- [Failure handling](docs/failure-handling.md) — recovery and resumability
- [Security and permissions](docs/security-and-permissions.md) — capabilities and approvals
- [Decisions](docs/decisions.md) — architectural decision log
- [Example run](docs/example-run.md) — invoice-ingestion walkthrough
- [Diagrams](docs/diagrams.md) — system and lifecycle views

