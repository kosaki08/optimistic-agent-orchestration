# Optimistic Agent Orchestration

A portable Agent Skill for deciding whether a bounded, reversible slice of dependent work can start before its upstream dependency is final.

`Start useful work early. Revalidate assumptions before making it authoritative.`

Extra concurrency only helps while the workflow can still validate and integrate what it produces. Past that point, speculative work moves the bottleneck downstream instead of removing it.

## What it does

The Skill helps an agent:

- classify each dependency as independent, soft, speculative, or hard
- start only a bounded slice whose assumptions and invalidation cost are written down
- treat starting the work and making its result authoritative as separate decisions
- revalidate upstream state before merge, send, publish, deploy, or another authoritative effect
- cap speculative work in progress by validation capacity rather than available agent slots
- switch to finish-first when the validation queue grows

## When to use it

Use it when unresolved upstream work, review, or contract decisions would normally block useful downstream work, but some isolated work may be repairable or discardable.

A client implementation may start against the observed revision of an API that is still under review, provided that the assumed contract is recorded, the slice remains isolated, and the final API is revalidated before integration.

## When not to use it

Do not use it to:

- parallelize ordinary independent tasks
- assign several agents to the same problem
- decide cross-task priority or scope
- bypass review, approval, permission, or external-effect gates
- justify speculation when waiting is cheaper than the added repair and validation work
- introduce a scheduler, registry, queue, state store, or plugin framework

## Install and invoke

The installable Skill directory is:

```text
skills/optimistic-agent-orchestration/
```

Copy it to wherever your client discovers skills. It follows the [Agent Skills specification](https://agentskills.io/specification) and needs no plugin system, task tracker, or workflow engine.

Ask for it in a prompt:

```text
Use the optimistic-agent-orchestration skill to decide which part of this blocked task can start now, and what has to be revalidated before merge.
```

Clients that support explicit invocation accept `$optimistic-agent-orchestration` in place of the name.

`evals/cases.md` records the behavior and trigger boundaries the Skill is checked against. It is not a runtime dependency.

## Prior art

This Skill does not claim a new concurrency-control algorithm. It applies established ideas from optimistic concurrency control and speculative execution to Coding Agent workflows:

- H. T. Kung and John T. Robinson, [On Optimistic Methods for Concurrency Control](https://doi.org/10.1145/319566.319567)
- David R. Jefferson, [Virtual Time](https://doi.org/10.1145/3916.3988)
- Cristian Tapus and Jason Hickey, [Distributed speculative execution for reliability and fault tolerance](https://doi.org/10.1007/s00446-008-0073-1)

The mapping to agent work is operational guidance: make assumptions explicit, keep speculative work reversible, revalidate before the result becomes authoritative, and stop adding speculative work once validation and integration are the bottleneck.

## License

Apache License 2.0. See [LICENSE](LICENSE).
