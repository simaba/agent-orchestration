# Multi-Agent Orchestration Patterns

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Discussions](https://img.shields.io/badge/Discussions-Join-7289da?style=flat-square&logo=github)](https://github.com/simaba/agent-orchestration/discussions)

A catalog of design patterns for orchestrating multiple AI agents, covering routing, delegation, validation, and failure handling in LLM-based multi-agent pipelines.

---

## Choose this repo when

Use this repository when you need **control-flow patterns** for multi-agent systems:

- how work is routed
- how subtasks are delegated
- how outputs are validated
- how retries, fallbacks, and escalation are structured

Do **not** start here if you need the broader oversight model. Use [`multi-agent-governance`](https://github.com/simaba/multi-agent-governance).

Do **not** start here if you need evaluation criteria and pass/fail dimensions. Use [`agent-eval`](https://github.com/simaba/agent-eval).

Do **not** start here if you want runnable behavior. Use [`agent-simulator`](https://github.com/simaba/agent-simulator).

---

## Practical artifacts

| Artifact | Use for |
|---|---|
| [`templates/orchestration-pattern-card.md`](templates/orchestration-pattern-card.md) | Documenting a reusable orchestration pattern with agents, flow, decisions, failure controls, logging, and evaluation criteria |
| [`examples/confidence-gated-fallback-pattern.md`](examples/confidence-gated-fallback-pattern.md) | Filled example for a confidence-gated fallback workflow |
| [`docs/structured-voting.md`](docs/structured-voting.md) | Deciding when majority vote is appropriate and what context, abstention, and escalation behavior it requires |

---

## Pattern catalog

### Routing patterns

| Pattern | When to Use | Description |
|---|---|---|
| **Classifier Router** | Multiple specialized agents | A classifier agent routes tasks to the most appropriate specialist |
| **Capability-Based Routing** | Agents with declared capabilities | Route tasks based on declared capabilities and task requirements |
| **Load-Balanced Round Robin** | Identical agent instances | Distribute tasks across replicas for throughput |
| **Priority Queue** | Mixed urgency tasks | Route high-priority tasks first and batch low-priority work |

### Delegation patterns

| Pattern | When to Use | Description |
|---|---|---|
| **Hierarchical Delegation** | Complex decomposable tasks | Orchestrator breaks tasks into subtasks and delegates execution |
| **Parallel Fan-Out** | Independent subtasks | Execute multiple subtasks in parallel and aggregate results |
| **Sequential Pipeline** | Order-dependent tasks | Pass outputs from one agent to the next in a fixed sequence |
| **Conditional Branch** | Decision-dependent workflows | Route to different agents based on intermediate results |

### Validation patterns

| Pattern | When to Use | Description |
|---|---|---|
| **Validator Agent** | High-stakes outputs | Separate agent reviews and approves outputs before use |
| **Majority Vote** | Bounded labels or structured verdicts | Aggregate independent responses only when all agents use the same explicit decision space |
| **Cross-Check** | Critical calculations | Two independent agents solve the same problem and divergence is flagged |
| **Confidence Gate** | Variable certainty | Only accept outputs above a threshold and escalate the rest |

### Failure-handling patterns

| Pattern | When to Use | Description |
|---|---|---|
| **Retry with Backoff** | Transient failures | Retry failed tasks with exponential backoff before escalation |
| **Fallback Agent** | Agent unavailability | Switch to a simpler fallback agent when the primary fails |
| **Human Escalation** | Unresolvable failures | Route to a human when retries and fallbacks are exhausted |
| **Circuit Breaker** | Cascading failures | Stop sending tasks to a failing agent until health checks recover |
| **Poison Pill Isolation** | Malformed inputs | Quarantine repeatedly failing inputs for review |

---

## Implementation examples

These snippets are **pseudocode**. They show control-flow choices, not production-ready authentication, retries, observability, authorization, data handling, or safety controls.

### Hierarchical delegation (Python pseudocode)

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class Task:
    description: str
    priority: str  # high | medium | low
    requires_human_review: bool = False

class OrchestratorAgent:
    def __init__(self, executor_agents, validator_agent):
        self.executors = executor_agents
        self.validator = validator_agent

    def execute(self, task: Task) -> dict[str, Any]:
        executor = self._select_executor(task)
        result = executor.run(task)

        if task.priority == "high" or task.requires_human_review:
            validation = self.validator.validate(task, result)
            if not validation.approved:
                return self._escalate_to_human(task, result, validation.reason)

        return result

    def _select_executor(self, task):
        return self.executors[0]

    def _escalate_to_human(self, task, result, reason):
        return {"status": "pending_human_review", "reason": reason}
```

### Majority vote for structured verdicts (Python pseudocode)

```python
from collections import Counter

def majority_vote(agents, task, allowed_labels, expected_votes=3):
    """Aggregate bounded labels; escalate on error, abstention, tie, or low agreement."""
    responses = [agent.run(task) for agent in agents[:expected_votes]]
    valid = [
        response
        for response in responses
        if response.get("status") == "valid"
        and response.get("label") in allowed_labels
    ]

    if len(valid) != expected_votes:
        return {
            "status": "pending_human_review",
            "reason": "insufficient valid votes",
            "valid_votes": len(valid),
            "expected_votes": expected_votes,
        }

    counts = Counter(response["label"] for response in valid)
    winner, winner_count = counts.most_common(1)[0]
    tied = list(counts.values()).count(winner_count) > 1
    agreement = winner_count / len(valid)

    if tied or agreement < 0.67:
        return {
            "status": "pending_human_review",
            "reason": "tie or insufficient agreement",
            "vote_counts": dict(counts),
            "agreement": agreement,
        }

    return {
        "status": "accepted_for_next_control",
        "label": winner,
        "vote_counts": dict(counts),
        "agreement": agreement,
        "note": "A vote is a workflow signal, not proof of correctness or approval.",
    }
```

Use this pattern only with an explicit label set and a documented independence assumption. For free-form answers, semantic aggregation and an accountable review path are usually more appropriate. See [`docs/structured-voting.md`](docs/structured-voting.md).

---

## Governance considerations

For each pattern, document:

- **who owns the orchestrator**
- **what gets logged**
- **where human oversight is triggered**
- **how failure is detected and handled**

See [`multi-agent-governance`](https://github.com/simaba/multi-agent-governance) for the companion governance framework.

---

## Related repositories

| Repository | What it adds |
|---|---|
| [`multi-agent-governance`](https://github.com/simaba/multi-agent-governance) | oversight model, trust tiers, accountability structure |
| [`agent-eval`](https://github.com/simaba/agent-eval) | measurable evaluation dimensions and scenarios |
| [`agent-simulator`](https://github.com/simaba/agent-simulator) | runnable implementation of bounded agent workflows |
| [`lean-ai-ops`](https://github.com/simaba/lean-ai-ops) | applied workflow automation in a separate domain |

*Maintained by [Sima Bagheri](https://github.com/simaba) · Connect on [LinkedIn](https://www.linkedin.com/in/simaba/)*