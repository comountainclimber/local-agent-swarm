# Security and permissions

Local-first reduces data sent to hosted models; it does not make autonomous code execution safe by default. Prompts, repository content, model output, shell commands, network calls, credentials, and generated diffs all cross trust boundaries.

## Capability model

Permissions should be explicit, composable, and scoped per run/task/agent:

- read repository;
- edit files within the assigned worktree;
- run named verification commands;
- run arbitrary shell commands;
- access the network or named endpoints;
- create commits and branches;
- merge changes;
- push or open pull requests.

Default to the minimum needed. The control plane enforces permissions at tool execution; instructions in a packet or model response cannot grant themselves more access. Separate inference-node credentials from frontier-provider credentials and from repository/hosting credentials.

## Human approval gates

Policies should normally require explicit approval for pushing, production actions, deployments, destructive database operations, secrets or security configuration changes, and expansion into sensitive scope. Autonomous merging is off by default. Deployment automation is outside v1.

## Isolation and data handling

- Worktrees isolate concurrent source edits, not hostile operating-system processes. Add stronger sandboxing later for untrusted commands.
- Send only relevant context to any provider; treat private source as sensitive even on a LAN.
- Redact tokens, `.env` contents, credentials, customer data, and secrets from prompts and logs.
- Store secrets in an OS keychain or dedicated secret facility, never SQLite plaintext or packets.
- Log provider/node selection, tool invocations, approvals, and state changes without logging secret values.
- Constrain subprocess environment, working directory, runtime, output size, and network according to policy.

## Threats to consider

Repository text, test output, dependency scripts, and issue descriptions can contain prompt injection. Model output can propose destructive commands or data exfiltration. A compromised inference endpoint can return malicious tool calls. Dependency installation can execute third-party lifecycle scripts. Review therefore combines hard enforcement, audit events, bounded packets, diff inspection, and human gates; model judgment is not the security boundary.

Static/security-oriented local reviewers can search for authorization, tenant isolation, secret exposure, injection, and unsafe migrations, but high-risk findings and final acceptance should escalate to frontier/human review.

