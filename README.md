# Multi-Agent Orchestration Patterns

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

A practitioner catalog of control-flow patterns for systems that route, delegate, validate, retry, fall back, escalate, and contain work across agents and tools.

The patterns are described as state and authority transitions rather than diagrams alone. A production workflow must define what evidence allows the next step, what external state may change, and how partial or uncertain execution is represented.

## Start here

| Artifact | Use it for |
|---|---|
| [`docs/control-flow-contract.md`](docs/control-flow-contract.md) | state, authority, evidence, retry, delegation, fallback, containment, and escalation semantics |
| [`templates/orchestration-pattern-card.md`](templates/orchestration-pattern-card.md) | documenting a reusable pattern |
| [`examples/confidence-gated-fallback-pattern.md`](examples/confidence-gated-fallback-pattern.md) | legacy worked example to review critically against the control-flow contract |
| [`docs/structured-voting.md`](docs/structured-voting.md) | deciding whether bounded voting is meaningful and how to handle abstention, ties, and escalation |

Use [`multi-agent-governance`](https://github.com/simaba/multi-agent-governance) for authority envelopes and system-level containment, [`agent-eval`](https://github.com/simaba/agent-eval) for evaluation validity, and [`agent-simulator`](https://github.com/simaba/agent-simulator) for runnable bounded behavior.

## Pattern selection questions

Before selecting a pattern, ask:

- What workflow state exists before and after each step?
- Which principal’s authority is being exercised?
- Which inputs are trusted data, untrusted content, or executable instruction?
- What evidence is required before output propagates?
- Can the step mutate external or persistent state?
- Is retry idempotent and safe after partial execution?
- How are disagreement, missing evidence, and abstention represented?
- What cancels queued or delegated work?
- Which failure moves the system to fallback, containment, or human decision?
- Can another reviewer reconstruct the full state and authority path?

## Routing patterns

### Rule or policy router

Use when routing can be derived from observable fields and explicit policy. Keep the rules versioned and test overlap, missing values, and default behavior.

**Failure modes:** stale policy, ambiguous categories, unsafe default route, and untrusted content influencing control fields.

### Learned classifier router

Use when labeled examples support a stable routing problem and misrouting has a bounded consequence.

**Required controls:** calibrated or otherwise validated decision rule, abstention path, drift monitoring, slice analysis, and a safe default.

Do not interpret a raw model score as universal probability of correct routing.

### Capability and authority router

Match task requirements to declared capability and permitted authority. Server-side tool and data controls must still enforce the boundary.

**Failure modes:** overstated capability, hidden permission differences, delegation that expands authority, and stale registry data.

### Priority and queue routing

Use for workload order, deadlines, and capacity. Preserve fairness, aging, cancellation, and incident override rules.

**Failure modes:** starvation, priority inflation, hidden queue delay, and duplicate execution after timeout.

## Delegation patterns

### Hierarchical delegation

An orchestrator decomposes work and assigns bounded subtasks.

Define subtask objective, data and tool authority, expected artifact, budget, stop condition, allowed delegation depth, and provenance back to the principal.

### Parallel fan-out

Use for genuinely independent read-only or isolated work. Specify snapshot consistency, cost limits, branch cancellation, missing-result handling, and whether shared state may be mutated.

### Sequential pipeline

Use when the output contract of one step is the input contract of the next. Validate schema, provenance, and instruction/data boundaries at every transition.

### Conditional branch

Use when the branch rule is explicit and testable. Record overlap, no-match, ambiguity, and policy-version behavior.

## Validation and evidence patterns

### Deterministic invariant check

Prefer executable validation for observable properties such as schema, permission, state, calculation, required citation, or prohibited tool use.

Passing an invariant check does not prove semantic quality beyond the checked property.

### Independent review

Use when judgment is required and the reviewer has sufficiently independent evidence or method.

A second agent is not automatically independent. Producer and reviewer may share the same model family, context, prompt assumptions, retrieval, tool outputs, or rubric.

### Evidence-gated propagation

Continue only when the evidence required for the next state is present and valid—for example provenance, contradiction status, policy result, action authorization, or human confirmation.

A calibrated confidence estimate may be one input for a defined task. “Confidence above 0.8” is not a general-purpose authorization rule.

### Cross-check and disagreement

Run separate methods or evidence paths and preserve divergence. Define whether disagreement triggers adjudication, more evidence, a bounded safer mode, or a hold.

### Structured vote

Use only for bounded labels in a shared decision space with a documented independence assumption. A vote is a workflow signal, not truth or approval. See [`docs/structured-voting.md`](docs/structured-voting.md).

## Failure and recovery patterns

### Retry with state verification

Retry only failures believed to be transient. Before retrying an external write, check whether the earlier attempt partially or fully succeeded. Use idempotency, bounded attempts, cumulative cost limits, and a terminal state.

### Capability-reducing fallback

A fallback should normally reduce authority or scope. Record quality differences, visible limitations, prohibited fallback contexts, and return-to-primary criteria.

### Circuit breaker and containment

Stop new work when a defined control or dependency state fails. Specify in-flight and queued actions, delegated work, degraded mode, reopening authority, and health evidence.

### Quarantine

Isolate malformed, adversarial, or repeatedly failing inputs and any derived memory or artifacts. Define access, retention, review, and safe re-entry.

### Human decision package

Escalate the current state, evidence, completed/partial/uncertain actions, options, and the exact decision the human is authorized to make. “Agent failed—please review” is not a usable escalation.

## Pseudocode example

The following illustrates state-aware evidence gating. It omits authentication, persistence, production authorization, observability infrastructure, and domain controls.

```python
from dataclasses import dataclass
from enum import Enum
from typing import Any

class Disposition(str, Enum):
    CONTINUE = "continue"
    HOLD = "hold"
    ESCALATE = "escalate"

@dataclass(frozen=True)
class EvidenceDecision:
    disposition: Disposition
    reasons: tuple[str, ...]
    validated_artifact: dict[str, Any] | None = None


def run_step(task, executor, validator, authorizer) -> dict[str, Any]:
    proposal = executor.propose(task)

    validation = validator.check(
        proposal,
        required_schema=task.output_schema,
        allowed_authority=task.authority_envelope,
        required_provenance=task.provenance_requirements,
    )
    if validation.disposition is not Disposition.CONTINUE:
        return {
            "status": validation.disposition.value,
            "reasons": list(validation.reasons),
        }

    authorization = authorizer.authorize(
        principal=task.principal,
        action=validation.validated_artifact,
        authority_envelope=task.authority_envelope,
    )
    if not authorization.approved:
        return {"status": "hold", "reasons": authorization.reasons}

    result = executor.execute(
        validation.validated_artifact,
        authorization_token=authorization.bound_token,
        idempotency_key=task.idempotency_key,
    )
    return executor.verify(result)
```

The important property is not the number of agents. It is the explicit separation of proposal, validation, authorization, execution, and verification.

## Pattern card requirements

For each pattern, document:

- intended and prohibited contexts;
- workflow states and transitions;
- principal, data, tools, and authority;
- input/output schema and provenance;
- validation method and independence assumptions;
- retry, timeout, cancellation, and partial-state behavior;
- fallback, containment, and recovery;
- observability and retention;
- evaluation scenarios;
- owner and invalidation triggers.

## Maturity and scope

This is a practitioner pattern catalog with pseudocode and public-safe artifacts. It is not a production orchestration library, formal verification method, or assurance that multi-agent behavior has been fully characterized.

## Related repositories

| Repository | Distinct role |
|---|---|
| [`multi-agent-governance`](https://github.com/simaba/multi-agent-governance) | authority envelopes, propagation governance, containment, and accountability |
| [`agent-eval`](https://github.com/simaba/agent-eval) | evaluation validity and decision semantics |
| [`agent-simulator`](https://github.com/simaba/agent-simulator) | runnable bounded retry, fallback, and escalation scenarios |

---

*Maintained by [Sima Bagheri](https://github.com/simaba).*
