---
name: optimistic-agent-orchestration
description: Use when unresolved upstream work, review, or contract decisions would normally block downstream coding or agent work, and decide whether a reversible slice may start early, which assumptions it depends on, and what must be revalidated before merge, send, publish, deploy, or another authoritative effect. Also use when upstream changes while dependent speculative work is in progress. Do not use for cross-task priority, scope conflicts, review finding disposition, plan changes, human escalation, ordinary parallelism between independent tasks, running several agents on the same problem, or prior-art research.
---

# Optimistic agent orchestration

Reduce critical-path waiting by starting a bounded, reversible slice of dependent work before its upstream dependency is final.

The unit of judgment is the dependency relationship, not the number of agents.

The caller is the current task or workflow that invokes this skill, whether a person, an agent, or an orchestration system drives it.

Core rule:

`Start useful work early. Revalidate assumptions before making it authoritative.`

The goal is not maximum concurrency. It is useful concurrency whose invalidation cost and final consistency can be bounded.

## Boundaries

- Preserve the caller's purpose, scope, authority, permission, task graph, review route, and acceptance criteria. This skill does not grant permission to start work or cross an external-effect gate.
- Use the task system, plan, issue, branch, worktree, review state, and validation commands already present. Do not create a new orchestrator, scheduler, registry, queue, state store, schema, watcher, memory service, or handoff format merely to apply this skill.
- Do not turn a single bounded task into a Feature or multi-agent workflow just to create parallel work.
- Do not use an LLM to poll deterministic state. Re-read authoritative state at a relevant checkpoint or use an existing event mechanism.
- Keep prior-art discovery, general work decomposition, executor selection, and learning from historical speculation results outside this skill.
- Keep cross-task priority, scope conflicts, review finding disposition, plan changes, and human escalation outside this skill; route them to the caller's existing cross-task coordination or authority process.

## 1. Fix the dependency and outcome

Identify:

- the outcome the caller actually requested, including a final authoritative or external effect only when that effect is in scope;
- the upstream work or decision that is unresolved;
- the downstream work that would normally wait;
- the exact information that downstream work needs from upstream;
- the current authoritative source and revision for that information.

Do not enlarge the outcome with a hypothetical later action. An unresolved decision that the requested work can inspect is not an upstream dependency unless that work needs the decision resolved to be correct.

Do not call an ordinary independent task speculative. If the current authority, revision, or unresolved decision cannot be established, stop rather than inventing an assumption.

## 2. Classify the relationship

Classify each relevant relationship, not the entire project.

| Relationship | Meaning | Start rule |
|---|---|---|
| Independent | Downstream correctness does not depend on the unresolved upstream result | Run with ordinary parallel orchestration |
| Soft | Preparation can start without predicting the upstream result | Start the independent preparation; validate normally |
| Speculative | Useful work can start only by assuming a checkable upstream result | Start only the bounded speculative slice and keep it out of the authoritative result until revalidated |
| Hard | Correct work cannot start safely, or the next action would cross an authoritative or irreversible effect | Wait or serialize |

Use this skill's assumption and revalidation flow for speculative relationships. Soft relationships need a boundary, but no invented upstream result.

## 3. Admit speculation only when it can pay

Allow speculative execution only when all of the following are true:

- waiting is materially extending the critical path;
- a useful downstream slice can run in isolation;
- the maximum wasted work is explicit and acceptable;
- the assumptions are few, concrete, and checkable;
- invalidation can lead to reuse, bounded repair, or discard without an unacceptable effect;
- required final validation can detect material incompatibility;
- speculative work-in-progress and depth remain bounded.

Do not speculate when any of the following applies:

- the upstream decision is changing frequently or has already invalidated this work repeatedly;
- the dependency is broad, hidden, or cannot be described well enough to revalidate;
- two workers would mutate the same authoritative decision or single-writer state;
- independently valid changes could violate a shared semantic invariant and no reliable combined validation exists;
- the speculative slice includes an irreversible or result-ambiguous external effect;
- the expected wait saved is small relative to the likely repair, discard, and validation cost.

File separation is not proof of semantic independence. Check shared contracts, acceptance criteria, policies, limits, and business rules.

### Bound WIP by validation capacity

Available workers are not the limiting resource. Additional speculative work is useful only when downstream validation can absorb it without extending total completion time.

Before increasing speculative WIP, compare:

- the expected upstream wait saved;
- the likely repair, discard, and validation cost;
- mechanical validation throughput and coverage;
- human review capacity for semantic or irreversible decisions.

Prefer deterministic revision, test, type, invariant, diff, and contract checks for mechanical questions. Reserve human review for semantic judgments or authority that cannot be delegated.

Keep speculative WIP at or below the capacity to validate and integrate completed work. The validator may be a tool, an agent reviewer, a human, or a combination. Human validation does not make speculation invalid, but a growing validation queue is a stop signal.

## 4. Record minimal working assumptions

Put the working record in the caller's existing task, issue, plan, branch description, or equivalent context. Do not leave it only in conversation memory, and do not introduce a dedicated persistent schema or artifact solely for this skill.

Record only what revalidation needs, and update `Current decision` at each checkpoint instead of writing a separate summary:

```text
Outcome and critical wait:
Dependency and observed upstream revision or state:
Relationship: independent | soft | speculative | hard
Assumptions: one to three predicted facts used by downstream work
Speculative slice: work allowed before upstream resolution
Waste and WIP bound: maximum acceptable wasted work, speculative WIP, and depth
Invalidation condition: upstream change or failed condition that stops the slice
Authoritative boundary and revalidation: state and validations to recheck before crossing
Current decision: start | wait | continue | repair | discard | serialize
```

Keep speculative assumptions with the dependent work. Read dependency revisions from their authoritative source. Re-read live state at validation time instead of copying it into new snapshots.

## 5. Execute without making the result authoritative

- Use the repository's existing isolation mechanism, such as a worktree, branch, sandbox, staged draft, or preview.
- Parallelize execution, not authority. Only one actor changes a given final decision, contract, or shared authoritative state at a time unless the system already has a safe concurrency protocol.
- Keep speculative chains shallow. Without an established policy and evidence for deeper speculation, do not start a speculative descendant from another speculative result.
- Keep an explicit speculative WIP cap separate from ordinary independent work.
- Stop at the declared boundary. Do not merge into an authoritative branch, send, publish, deploy, freeze a contract, or perform another authoritative effect from an unvalidated assumption.
- Do not add managers, status writers, or monitoring sessions in proportion to the number of workers.

If upstream changes before the normal checkpoint, pause the affected slice and move directly to impact assessment.

## 6. Revalidate and resolve conflicts

At upstream resolution and immediately before the result becomes authoritative, re-read authoritative state. Compare it with the recorded assumptions and observed revision.

Check, as applicable:

- the final upstream result and revision;
- each recorded assumption;
- shared semantic invariants and combined acceptance criteria;
- current authority and permission for the final action;
- required tests, checks, review, and approval on the integrated result;
- whether the external effect has already occurred.

Choose one result:

| Result | Condition | Action |
|---|---|---|
| Continue | Assumptions still hold | Integrate and run the required final validation |
| Repair | A bounded subset changed and unaffected work is identifiable | Preserve unaffected work, repair the affected slice, then rerun all required final validation |
| Discard or replan | The plan or core contract is invalid | Discard the speculative result or restart from current state |
| Serialize | Conflict or invalidation is repeated, opaque, or too costly | Stop speculating and wait for the dependency |

Do not treat semantic repair by an agent as proof of correctness. Use deterministic tests and invariant checks where they exist, then apply the workflow's required review.

If an external action returns an unknown outcome, do not assume failure and retry. Reconcile with the authoritative external system first, unless an existing idempotency mechanism already makes the retry safe.

## 7. Cross the commit barrier

The point where speculative work becomes authoritative or externally consequential is a logical boundary; this skill calls it the commit barrier. It does not imply a specific Git operation, review process, or human approval step, and it does not prescribe which validation or approval the caller's workflow requires. It only requires that the recorded assumptions are rechecked against the final upstream state before crossing; use the caller's existing policy for everything else.

Cross the barrier only after the final upstream state, the dependent result, and all required evidence refer to compatible revisions.

The barrier includes any action that makes the result authoritative or externally consequential, including:

- merge into an authoritative branch, or contract freeze;
- send or publish;
- deploy or production mutation;
- payment, permission, credential, or destructive change.

Compensation is not the same as rollback. A correction, cancellation, or refund does not recreate a world in which the first effect never happened.

## Return the decision

Return the working record from section 4 with `Current decision` updated, in the caller's existing workflow. Do not persist it as a new artifact unless the existing workflow requires one.

## Stop conditions

Stop optimistic execution and return to the caller when:

- authority, dependency identity, or current revision is inconsistent or unavailable;
- the speculative slice would exceed its waste, time, token, or WIP bound;
- a new irreversible effect or permission expansion appears;
- a shared invariant cannot be validated;
- upstream changes alter the accepted purpose, scope, or contract;
- the commit-barrier validation or review queue is already above the workflow's effective capacity; finish and validate existing speculative work before starting more;
- repeated invalidation makes serialization cheaper or safer.

For reusable behavior and trigger checks, use [evals/cases.md](evals/cases.md).
