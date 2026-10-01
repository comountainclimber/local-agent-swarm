# Diagrams

## High-level system architecture

```mermaid
flowchart LR
    U[Human] --> F[Frontier planner / reviewer]
    F --> O[Orchestrator]
    O --> DB[(SQLite state + events)]
    O --> S[Scheduler / router]
    O --> W[Worktree manager]
    W --> A1[Agent process: task A]
    W --> A2[Agent process: task B]
    A1 --> T[Local tools, tests, services]
    A2 --> T
    A1 --> P[Provider abstraction]
    A2 --> P
    P --> N1[Magnitude node 1]
    P --> N2[Magnitude node 2]
    P --> NX[Other compatible provider]
    O --> V[Verification + integration]
    V --> F
    F --> U
```

The agent processes and repository tools remain on the control plane; only inference requests cross to nodes.

## Task DAG

```mermaid
flowchart TD
    C[Extraction contract] --> E[Inbound email]
    C --> P[PDF extraction]
    C --> PJ[Project matching]
    C --> M[Material matching]
    PJ --> UI[Pending-review UI]
    M --> UI
    E --> IT[Integration tests]
    P --> IT
    UI --> IT
    IT --> R[Adversarial / final review]
```

## Worker execution lifecycle

```mermaid
stateDiagram-v2
    [*] --> Leased
    Leased --> Inspecting
    Inspecting --> Editing
    Editing --> Verifying
    Verifying --> Editing: failed, retryable, within limits
    Verifying --> SelfReview: required checks pass
    SelfReview --> Editing: issue found
    SelfReview --> Completed: criteria satisfied
    Inspecting --> Blocked: dependency / decision missing
    Editing --> Failed: crash or limit
    Verifying --> Failed: exhausted
    Completed --> Review
    Review --> Accepted
    Review --> RepairRequired
    RepairRequired --> [*]: orchestrator creates repair packet
    Accepted --> [*]
    Blocked --> [*]
    Failed --> [*]
```

## Multi-node inference routing

```mermaid
flowchart TD
    Q[Agent inference request] --> F{Filter eligible nodes}
    F -->|health + model + capability + capacity| R{Rank policy}
    R -->|least loaded / preference| N2[Node 2: coder]
    R --> N3[Node 3: coder]
    R --> N4[Node 4: reviewer]
    N2 --> X{Result}
    N3 --> X
    N4 --> X
    X -->|success| A[Return normalized event stream]
    X -->|transient failure| B[Backoff and reassign]
    X -->|OOM| C[Lower concurrency or different model/node]
    X -->|repeated semantic failure| E[Escalate model or frontier]
```

## Feature execution lifecycle

```mermaid
sequenceDiagram
    actor Human
    participant Frontier
    participant Orchestrator
    participant Worker
    participant Node as Inference node
    participant Review
    Human->>Frontier: Goal and constraints
    Frontier->>Frontier: Inspect repository and resolve ambiguity
    Frontier->>Orchestrator: DAG + versioned work packets
    Orchestrator->>Worker: Worktree, packet, permissions
    loop Edit / verify within limits
        Worker->>Node: Compatible HTTP inference request
        Node-->>Worker: Model stream / tool call
        Worker->>Worker: Execute locally and capture evidence
    end
    Worker-->>Orchestrator: Structured result + diff + evidence
    Orchestrator->>Review: Packet, result, evidence
    Review-->>Orchestrator: Accept, repair, or escalate
    opt Follow-up or repair approved
        Orchestrator->>Worker: New bounded packet
    end
    Orchestrator->>Orchestrator: Integrate and run aggregate checks
    Orchestrator->>Frontier: Integrated diff and evidence
    Frontier-->>Human: Final review and residual risks
    Human->>Orchestrator: Approve or reject merge/push
```

