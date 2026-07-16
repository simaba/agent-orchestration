# Orchestration Control-Flow Contract

An orchestration pattern should specify more than the order of agent calls. It should define the state, authority, evidence, and failure semantics at every transition.

## Transition record

For each step or edge in the workflow, record:

| Field | Question |
|---|---|
| State | What workflow state exists before and after the transition? |
| Principal | On whose behalf is the action taken? |
| Input contract | Schema, provenance, and trusted versus untrusted fields? |
| Authority | What data, tool, and external action may occur? |
| Preconditions | Which evidence or control must pass? |
| Decision rule | Deterministic rule, calibrated estimate, human authorization, or other? |
| Output contract | Schema, provenance, uncertainty, and disposition? |
| Timeout / cancellation | What happens when the step does not complete? |
| Retry semantics | Is retry safe, idempotent, bounded, and state-aware? |
| Failure state | How are partial, invalid, and uncertain results represented? |
| Escalation | Who receives what evidence and which decision can they make? |
| Observability | Which versions, tool calls, state changes, and dispositions are recorded? |

An edge that cannot describe its failure state is likely to hide it in an exception, retry loop, or free-form message.

## Evidence-gated propagation

Do not authorize downstream action merely because an agent reports high confidence.

A propagation gate should evaluate the evidence needed for the next state, such as:

- required fields and schema validity;
- source and data provenance;
- contradiction or missing-evidence status;
- deterministic policy or permission checks;
- evaluator result and limitations;
- action authority and confirmation;
- freshness and version compatibility;
- explicit abstention or escalation state.

A numerical confidence may be one input when it is calibrated for the relevant task and population. It should not be treated as a universal correctness probability.

## Validation independence

A validator can fail with the producer through shared:

- model family or provider;
- system prompt and examples;
- retrieval corpus;
- tool results;
- evaluator rubric;
- context or memory;
- training-data bias;
- optimization objective.

For consequential decisions, specify the required independence:

- deterministic invariant check;
- separately implemented rule;
- different evidence source;
- blinded human review;
- independent model or reviewer with calibrated disagreement handling;
- formal authorization by an accountable principal.

A second agent call is not automatically an independent control.

## Retry semantics

Before adding retry, answer:

- Which failures are plausibly transient?
- Is the operation read-only, idempotent, or protected by an idempotency key?
- Can a prior attempt have partially changed external state?
- Is the retry using the same input and method, or changing the plan?
- What evidence determines success?
- How is cost, latency, and cumulative authority bounded?
- When does retry stop and move to containment or human decision?

Do not retry an uncertain external write until the resulting state is checked. Retrying can duplicate messages, purchases, records, deployments, or deletions.

## Fan-out and aggregation

Parallelism can increase speed and coverage, but it also creates:

- duplicated external actions;
- inconsistent snapshots;
- rate and cost spikes;
- shared-resource contention;
- conflicting outputs;
- cancellation and cleanup complexity;
- correlated error presented as consensus.

Define whether fan-out branches are read-only, isolated, independently sampled, or allowed to mutate shared state. Aggregation should preserve disagreement and missing branches.

## Delegation

A delegated task should include:

- bounded objective and stop condition;
- data and tool authority;
- allowed sub-delegation depth and fan-out;
- expected artifact and evidence;
- cost and time budget;
- cancellation token or equivalent;
- provenance back to the original principal;
- behavior when the task cannot be completed safely.

Delegation must not expand authority unless a separate authorization grants it.

## Fallback

A fallback is safe only when it has an explicit capability and quality contract.

Record:

- failures that trigger fallback;
- whether the fallback has narrower or different authority;
- quality and limitation differences visible to users;
- data and state handoff;
- conditions that prohibit fallback;
- whether the fallback result requires additional review;
- recovery and return-to-primary criteria.

A simpler model can be more predictable while still being less capable or differently unsafe.

## Circuit breaker and containment

A circuit breaker should define:

- monitored failure signal and time window;
- state transitions: closed, open, half-open, disabled, or domain-specific equivalents;
- effect on in-flight and queued work;
- fallback or degraded mode;
- authority to reopen;
- health-check evidence;
- manual override and incident ownership;
- external state verification.

Stopping new requests does not automatically contain delegated or already authorized actions.

## Human escalation

Escalation should provide a decision package, not an opaque failure notice.

Include:

- original request and intended outcome;
- current workflow state;
- completed, partial, failed, and uncertain actions;
- relevant inputs, evidence, and tool trace;
- disagreement or missing evidence;
- available options and consequences;
- decision the human is authorized to make;
- timeout and default behavior;
- correction and follow-through path.

## Pattern review outcome

For each proposed orchestration pattern, conclude:

- where it is appropriate;
- prohibited or unsuitable contexts;
- required authority and isolation;
- required evidence and validation independence;
- failure and partial-state handling;
- monitoring and containment;
- evaluation scenarios;
- owner and change triggers.
