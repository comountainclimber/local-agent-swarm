# Architecture

## Boundaries

Local Agent Swarm coordinates software-engineering work; it does not serve models or replace git. Magnitude or another compatible server owns model loading and hardware optimization. Git owns source history. Coding-agent runtimes such as Codex CLI or OpenCode own the edit/tool loop. The orchestrator connects these systems and records their outcomes.

Three locations must remain independent:

- **Agent runtime:** the process interpreting model responses and invoking tools.
- **Repository/worktree:** the filesystem in which tools operate.
- **Inference:** the process and hardware producing model responses.

In the initial design, the first two are on the control plane. Inference may be local or remote. No source checkout is required on an inference-only node.

## Components

| Component | Responsibility | Explicitly does not own |
|---|---|---|
| Frontier planner/reviewer | Investigate, clarify, decompose, approve repairs, final review | Routine implementation loops |
| Orchestrator | Run lifecycle, task DAG, persistence, approvals, integration coordination | Model serving or custom git behavior |
| Scheduler/router | Find ready tasks; select eligible provider/node/model and retry policy | Business-level task decomposition |
| Worktree manager | Create branch/worktree, validate cleanliness, archive logs, remove safely | Automatic merge approval |
| Agent adapter | Launch a coding runtime with packet, provider settings, and limits | Inference implementation |
| Inference provider | Normalize streaming requests/events | Tool execution or repository access |
| Node registry | Endpoint, model, capability, capacity, and health metadata | Network transport configuration |
| Verification runner | Execute declared checks and capture evidence | Deciding product correctness alone |
| Reviewer | Compare diff and evidence against packet; propose repairs | Silently expanding task scope |
| State/event store | Durable run state and append-only operational evidence | Source-of-truth code history |

## Provider boundary

The rest of the system depends on a normalized streaming contract, never on Magnitude-specific calls:

```ts
interface InferenceProvider {
  chat(request: ChatRequest): AsyncIterable<ChatEvent>
}

type ChatEvent =
  | { type: "text-delta"; text: string }
  | { type: "tool-call"; id: string; name: string; arguments: unknown }
  | { type: "usage"; inputTokens?: number; outputTokens?: number }
  | { type: "completed"; finishReason: string }
  | { type: "error"; code: string; retryable: boolean; message: string };
```

Likely adapters are `MagnitudeProvider`, `OpenAIProvider`, `AnthropicProvider`, and a generic `OpenAICompatibleProvider`; MLX may be added later. Provider adapters normalize authentication, request shape, streaming, errors, usage, and cancellation. They do not decide routing.

## Node registry

```ts
interface Node {
  id: string;
  endpoint: string;
  models: string[];
  capabilities: { coding: boolean; reasoning: boolean; vision: boolean };
  status: "idle" | "busy" | "offline" | "degraded";
  activeSessions: number;
  maxSessions: number;
  hardware?: { chip?: string; memoryGB?: number };
}
```

Runtime health observations should be stored separately from desired configuration so a failed probe does not erase operator intent. Nodes are disposable: loss of a node may fail an attempt, but must not lose the task.

## Task and state model

A feature becomes a directed acyclic graph (DAG) of immutable work-packet revisions. A task becomes ready only when all required dependencies have an accepted result. Execution creates an **attempt**; retries create new attempts rather than overwriting history.

SQLite is sufficient initially. Candidate entities:

- `runs`, `features`, `tasks`, `task_dependencies`
- `workers`, `inference_nodes`, `attempts`
- `verification_results`, `artifacts`, `events`

Every material transition appends an event: packet issued, attempt started, node selected, command run, result reported, review requested, repair approved, integration accepted, or approval denied. Tables may cache current state, while events preserve the audit trail.

## Ownership and integration

Each task receives a dedicated branch, worktree, agent process, log stream, result, and verification plan. The orchestrator never lets two tasks edit the same worktree. Integration consumes accepted task branches in dependency order, reruns appropriate checks in an integration worktree or the primary checkout, and stops on conflicts or failed gates. Merging, pushing, and opening pull requests are separate permissioned actions.

## Control loop

1. A user supplies a goal and constraints.
2. The frontier planner inspects the repository and produces a task DAG plus packets.
3. The orchestrator validates packet shape, dependency acyclicity, permissions, and verification commands.
4. The scheduler leases ready tasks and suitable inference capacity.
5. Agent processes iterate inside isolated worktrees.
6. Verification evidence and structured results are persisted.
7. Independent reviewers accept, reject, or propose bounded repair work.
8. Workers may propose follow-ups; the orchestrator or human approves scope expansion.
9. Accepted branches integrate; aggregate checks run.
10. A frontier model performs final cross-cutting review and a human controls the final merge/push boundary.

See [diagrams](diagrams.md), [scheduling](scheduling.md), and [failure handling](failure-handling.md).

