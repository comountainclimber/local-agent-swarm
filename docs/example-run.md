# Example run: invoice PDF ingestion and material matching

This walkthrough shows how “Add invoice PDF ingestion and material matching to a contractor SaaS” becomes controlled, parallel work.

## 1. Investigation and clarification

The user submits the feature and business constraints. The frontier planner inspects existing email ingestion, document storage, tenant/project models, material catalog, review UI, tests, and repository conventions. It asks only decisions that materially affect design—for example, whether ambiguous project matches must always enter manual review—and records the answers.

## 2. DAG and work packets

The planner proposes:

| Task | Depends on | Main outcome |
|---|---|---|
| `extraction-contract` | — | Canonical extracted invoice/line-item types |
| `inbound-email` | extraction contract | Store attachment and create ingestion record |
| `pdf-extraction` | extraction contract | Convert supported PDFs into canonical data |
| `project-matching` | extraction contract | Tenant-scoped project candidates |
| `material-matching` | extraction contract | Deterministic catalog candidates/confidence |
| `pending-review-ui` | project + material contracts | Human correction and confirmation flow |
| `integration-tests` | all implementation tasks | End-to-end ingestion and isolation scenarios |
| `adversarial-review` | all above | Security, tenant boundary, and failure review |

Each packet names relevant files/symbols, invariants, non-goals, acceptance criteria, commands, permissions, and expected output. The material packet, for example, forbids changes to OCR and authentication and requires deterministic top-three results with no cross-tenant candidate.

## 3. Dispatch and inference routing

The orchestrator validates the DAG and creates branches/worktrees for the ready contract task. Once accepted, independent packets become ready. Agent processes all run on Mac 1 in distinct worktrees:

- Mac 2's coding model serves inbound-email inference.
- Mac 3's stronger coding model serves PDF extraction.
- Mac 4 serves project/material tasks sequentially under its session limit.
- Mac 1 continues running tools, tests, Docker, git, and optionally a local reviewer model.

Only prompt context and responses cross the network. Remote Macs never edit source.

## 4. Iteration and evidence

Each worker inspects its packet and code, edits, runs targeted checks, diagnoses failures, reruns, inspects its diff, and emits a structured result. Commands, exit states, concise summaries, logs, selected model/node, duration, and retry events attach to the attempt.

Suppose the material worker discovers seven serializers rely on an older candidate type. Because those call sites are outside its packet, it proposes `update-material-candidate-serializers` with the affected symbols. The orchestrator checks overlap and dependencies; the frontier planner approves and specifies a repair packet. Scope grows deliberately rather than accidentally.

## 5. Review and failure handling

Reviewer workers compare each final diff with its packet and evidence. They search for missing negative tests, undocumented behavior, and tenant-isolation errors. If Mac 3 sleeps mid-request, that attempt records a transient node failure; its worktree remains intact and the task is retried on an eligible node. If a worker repeatedly misunderstands the extraction contract, it escalates to a stronger local model or frontier clarification rather than looping indefinitely.

The adversarial reviewer finds that a stored attachment lookup lacks a tenant predicate. It returns `repair-required` with evidence. A narrow repair task and regression test run before the original task can be accepted.

## 6. Integration and final control

Accepted branches integrate in dependency order. Conflicts stop integration and produce a repair/reconciliation packet; no worker silently overwrites another result. The control plane runs aggregate integration, type, lint, and build checks on the combined source state.

The frontier model then reviews the integrated diff, test evidence, assumptions, concerns, and architectural fit. The orchestrator presents the final outcome and any residual risks. A human approves the merge and, separately, any push or pull-request action allowed by policy.

The event history now explains what was requested, how it was decomposed, which models/nodes worked on it, what changed, which checks ran, what failed, why follow-up work appeared, and who authorized integration.

