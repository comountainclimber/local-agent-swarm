# Work packets

A work packet is the contract between planning and execution. Small local models perform best when they do not need to rediscover product intent, infer forbidden scope, or guess what “done” means. Specific packets concentrate inference on the edit-and-verify loop and make results comparable across models.

## Contract

```ts
interface WorkPacket {
  id: string;
  revision: number;
  title: string;
  objective: string;
  context: {
    architectureSummary: string;
    relevantFiles: string[];
    relevantSymbols?: string[];
    conventions: string[];
    priorDecisions?: string[];
  };
  constraints: string[];
  nonGoals: string[];
  acceptanceCriteria: string[];
  verification: { commands: string[]; tests?: string[] };
  dependencies?: string[];
  permissions?: string[];
  limits?: { maxAttempts?: number; maxDurationMinutes?: number };
  expectedOutput: {
    summary: boolean;
    changedFiles: boolean;
    testsRun: boolean;
    assumptions: boolean;
    concerns: boolean;
  };
}
```

Packets should identify relevant starting points without pretending the planner knows every affected file. Acceptance criteria describe observable behavior. Constraints and non-goals define the change boundary. Verification commands must be precise enough to run without interpretation and safe under the packet's permissions.

## Example

```yaml
id: invoice-material-matching
revision: 1
title: Add deterministic invoice material matching
objective: Match extracted invoice line items to tenant-scoped catalog materials.
context:
  relevantFiles: [src/materials/matcher.ts, src/api/invoices.ts]
  conventions: [Use repository normalization helpers, Preserve tenant scoping]
constraints:
  - Exact SKU match has confidence 1.0.
  - Normalized exact name has confidence at least 0.95.
  - Never auto-accept below 0.90.
  - Return at most three candidates with deterministic ordering.
nonGoals: [Change OCR extraction, Change authentication, Assign projects]
acceptanceCriteria:
  - Exact, fuzzy, and no-match cases are covered.
  - A material from another tenant is never returned.
  - Existing API response compatibility is preserved.
verification:
  commands: [npm test -- matcher, npm run typecheck]
dependencies: [invoice-extraction-contract]
```

## Packet quality checklist

A packet is dispatchable when:

- its objective has one coherent outcome;
- dependencies name outputs, not merely related tasks;
- context points to evidence the worker can inspect;
- acceptance criteria are testable and non-contradictory;
- allowed scope and forbidden scope are both clear;
- commands are proportionate, reproducible, and authorized;
- the expected result format and stop conditions are explicit.

Packets are versioned. A clarification produces a new revision and event; it must not silently mutate an active attempt.

## Worker result

```ts
interface WorkerResult {
  taskId: string;
  attemptId: string;
  packetRevision: number;
  status: "completed" | "blocked" | "failed";
  summary: string;
  changedFiles: string[];
  verification: Array<{
    command: string;
    status: "passed" | "failed" | "not-run";
    outputSummary?: string;
    artifactRef?: string;
  }>;
  assumptions: string[];
  concerns: string[];
  followUpTasks?: ProposedTask[];
}
```

“Completed” means the declared acceptance criteria appear satisfied and required verification passed—not merely that files changed. “Blocked” identifies a concrete missing dependency, permission, decision, or environmental condition. “Failed” means the bounded attempt ended without a valid result.

Follow-up tasks are proposals, not authorization. For example, a worker discovering seven serializer call sites should report a bounded migration task instead of editing them outside scope. The planner/orchestrator checks duplication, dependencies, risk, and human approval rules before adding it to the DAG.

