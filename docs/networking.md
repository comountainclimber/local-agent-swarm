# Networking

## Endpoint-only abstraction

The orchestrator addresses inference servers by URL, for example:

```text
http://127.0.0.1:10100/inference/v1
http://192.168.1.102:10100/inference/v1
http://192.168.1.103:10100/inference/v1
```

Requests use ordinary HTTP and provider-compatible streaming. The transport may be Wi-Fi LAN, Ethernet, Thunderbolt Bridge, or Tailscale/private networking; routing and scheduling logic are unchanged. Thunderbolt Bridge is an optional fast, stable IP link—not a special API, shared-memory mechanism, or requirement.

## Initial topology

The control plane initiates connections to inference nodes. Inference-only nodes do not need repository access and do not initiate tool calls against the control plane. This reduces setup and limits the trust boundary: prompts and model responses cross the network, while source files move only when included in model context.

Each configured node should have a stable ID and endpoint, advertised models/capabilities, concurrency limit, and health-check method. Connection, response, and total-request timeouts are distinct. Streaming connections require cancellation and detection of stalled streams.

## Security baseline

- Prefer a trusted LAN or authenticated private overlay; do not expose local inference endpoints directly to the public internet.
- Bind servers to the narrowest practical interface and use host firewalls.
- Use TLS and endpoint authentication whenever traffic crosses an untrusted network.
- Store credentials outside packets, logs, and repository files.
- Redact secrets before sending repository context and record which provider/node received a request.
- Allowlist endpoints to prevent packets or model output from redirecting inference traffic.

Discovery can remain static configuration in early phases. Automatic discovery, service meshes, and custom distributed protocols are unnecessary for v1.

## Manual validation before implementation

Phase 0 should end with a documented operator test: start Magnitude on a remote Mac, confirm reachability from the control plane, list/identify a model, issue a small streaming request, test cancellation, and observe behavior when the node stops. Exact commands belong in implementation-era operator documentation because the upstream API may evolve.

