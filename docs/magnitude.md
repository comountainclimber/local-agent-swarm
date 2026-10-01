# Magnitude integration

Magnitude is the preferred first local inference engine for Apple Silicon. It is responsible for model loading, inference, hardware optimization, and serving compatible HTTP APIs. This project does not fork Magnitude, modify it, or build competing serving infrastructure.

## Intended relationship

A Magnitude process runs on one or more Macs. A coding-agent process on the control plane sends model requests over HTTP, receives text/tool-call output, and executes tools locally in its assigned worktree. Multiple agents may use a node when its memory and performance permit, though the initial operational default is one heavy session per machine.

The `MagnitudeProvider` translates the normalized provider contract into the supported OpenAI- or Anthropic-compatible API shape. Provider-specific configuration stays at this edge:

- endpoint and authentication;
- model identifier mapping;
- streaming event translation;
- timeouts and cancellation;
- error/usage normalization.

The task graph, worker lifecycle, verification, and repository code must contain no Magnitude assumptions.

## Why preferred, not embedded

Magnitude offers a short path to networked Apple Silicon inference without requiring Swift, Metal code, RDMA, or a custom protocol. Treating it as replaceable keeps the architecture open to generic OpenAI-compatible servers, MLX-based servers, Linux NVIDIA/AMD hosts, cloud workers, or frontier APIs.

## Validation questions

Before Phase 1, manually verify the deployed Magnitude version's endpoint paths, model discovery, streaming/tool-call behavior, cancellation, concurrency behavior, error shapes, context limits, and remote binding/authentication options. Record observed capabilities in node configuration rather than assuming every compatible endpoint behaves identically.

No multi-machine model sharding is planned for v1. Four Magnitude nodes are four independent brains behind one scheduler.

