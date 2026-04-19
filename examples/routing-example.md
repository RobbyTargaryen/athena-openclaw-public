# Adaptive Routing Engine Logic

The routing engine dynamically assigns incoming prompts to the optimal node and model combination. This ensures low latency and cost-efficiency while preserving high reasoning capabilities for complex tasks.

## Routing Pseudo-Code

The core logic evaluates task complexity, keyword intent, cost priority, and live node health before dispatching the payload to the OpenClaw execution layer.

```python
def determine_execution_route(prompt: str, source: str, system_health: dict) -> dict:
    """
    Evaluates semantic intent and system health to route execution.
    Tradeoffs: Prioritizes local latency over cloud reasoning unless explicitly escalated.
    """
    prompt_lower = prompt.lower()
    length = len(prompt)
    
    # 1. Detect Interaction Mode
    # Interactive sessions inherently require higher reasoning for debugging.
    is_interactive = (source == "tui" or source == "interactive")
    
    # 2. Semantic Intent Classification
    if any(k in prompt_lower for k in ["optimize", "latency", "bottleneck"]):
        task_type = "optimization"
    elif any(k in prompt_lower for k in ["code", "debug", "script", "deploy"]):
        task_type = "coding"
    elif any(k in prompt_lower for k in ["plan", "architecture", "strategy"]):
        task_type = "planning"
    else:
        task_type = "operations"
        
    # 3. Complexity & Escalation Scoring
    complexity = "high" if length > 500 or task_type in ["planning"] else "medium" if task_type in ["coding", "optimization"] else "low"
    
    cost_priority = "high" if any(k in prompt_lower for k in ["production", "critical", "outage"]) else "normal"
    
    # 4. Agent and Model Mapping
    # Logic prioritizes keeping data local and avoiding cloud costs.
    if complexity == "low" and not is_interactive:
        agent = "ambient-agent"
        model = "local/fast-inference-8b"
    elif task_type == "coding":
        agent = "coder-agent"
        model = "local/specialized-coder-14b"
    elif task_type == "optimization":
        agent = "planner-agent"
        model = "local/reasoning-7b" 
    elif task_type == "planning" or cost_priority == "high":
        # Escalation to cloud occurs only here due to high cost priority or extreme complexity.
        agent = "planner-agent"
        model = "cloud/premium-reasoning-model"
    else:
        agent = "ambient-agent"
        model = "local/fast-inference-8b"
        
    # 5. Health-Aware Fallback
    # If the targeted local node is degraded, fail over to the next capable local node.
    if "local" in model and system_health.get(agent) == "degraded":
        model = "local/fallback-inference-8b"
        
    return {
        "task_type": task_type,
        "complexity": complexity,
        "agent": agent,
        "selected_model": model
    }
```
