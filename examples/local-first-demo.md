# Local-First Execution Demo

This document demonstrates the execution flow when an operator requests a system diagnostic. It highlights how the orchestration layer classifies intent, maintains privacy by routing to local models, and executes tools securely.

## Interaction Trace

**Operator:** *"Diagnose why the memory caching service is failing on node-02."*

### 1. Intent Classification (Adaptive Routing Engine)

The routing proxy intercepts the prompt before it hits the OpenClaw execution gateway.

```json
{
  "mode": "ambient",
  "task_type": "diagnostics",
  "complexity": "medium",
  "cost_priority": "normal",
  "agent_route": "operations-agent",
  "model_selection": "local/reasoning-7b"
}
```
*Rationale:* The system identified the keyword "diagnose" and assigned it to the `operations-agent` using a local reasoning model (`local/reasoning-7b`). It avoided escalating to a cloud model, preserving execution budget.

### 2. Execution Layer (OpenClaw Gateway)

The task is forwarded to the designated agent. The agent utilizes local execution tools (like `exec` or `read`) to probe the target node.

```bash
# Agent internally executes:
$ ssh node-02 "systemctl status memory-cache.service --no-pager && journalctl -u memory-cache.service -n 20"
```

### 3. Synthesis & Response

The local model synthesizes the raw log output and returns a structured, actionable response to the operator.

**System Response:**
> **Diagnostic Report for node-02:**
> The `memory-cache.service` is failing to start. 
> 
> **Root Cause:** 
> Journal logs indicate a `TimeoutError: timed out` when attempting to bind to the local socket, likely due to a stale lock file preventing service initialization.
> 
> **Recommended Remediation:**
> Execute a cleanup script to clear `/tmp/cache.lock` and restart the service. Would you like me to execute this remediation?
