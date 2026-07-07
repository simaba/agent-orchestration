# Structured Voting Boundaries

Majority vote is useful only when each response can be reduced to the **same bounded decision space**. It is not a safe way to aggregate free-form long-form answers by exact string matching.

## Use voting when

- each agent selects from the same explicit labels or structured verdicts
- each response includes a status such as `valid`, `abstain`, or `error`
- the orchestrator records agent/version, task version, input identifier, and response count
- a tie, insufficient response count, or abstention triggers a fallback or human review

## Do not use voting when

- answers are materially different but semantically similar natural-language text
- the decision is safety-critical, legally consequential, financially irreversible, or otherwise requires an accountable review path
- agents share the same failure mode, training source, tool dependency, or prompt path and are incorrectly treated as independent evidence

## Minimum vote record

Record the declared options, participating agents and versions, expected vote count, actual valid vote count, abstentions/errors, winning label, agreement rate, and escalation decision.

A vote can be one signal in a workflow. It is not evidence of truth, correctness, safety, or release approval on its own.
