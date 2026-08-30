# Optimistic Agent Orchestration

`Start useful work early. Revalidate assumptions before making it authoritative.`

A portable Agent Skill for deciding whether a bounded, reversible slice of dependent work can start before its upstream dependency is final.

The goal is not maximum concurrency. It is to reduce critical-path waiting without creating more repair, review, and integration work than the workflow can absorb.

## What it does

The Skill helps an agent:

- classify each dependency as independent, soft, speculative, or hard;
- start only a bounded slice whose assumptions and invalidation cost are explicit;
- keep execution separate from authority;
- revalidate upstream state before merge, send, publish, deploy, or another authoritative effect;
- cap speculative work in progress by validation capacity rather than available agent slots;
- switch to finish-first when the validation queue grows.

## When to use it

Use it when unresolved upstream work, review, or contract decisions would normally block useful downstream work, but some isolated work may be repairable or discardable.

For example, a client implementation may start against the observed revision of an API that is still under review, provided that the assumed contract is recorded, the slice remains isolated, and the final API is revalidated before integration.

## When not to use it

Do not use it to:

- parallelize ordinary independent tasks;
- assign several agents to the same problem;
- decide cross-task priority or scope;
- bypass review, approval, permission, or external-effect gates;
- justify speculation when waiting is cheaper than the added repair and validation work;
- introduce a scheduler, registry, queue, state store, or plugin framework.

## Install and invoke

The installable Skill directory is:

```text
skills/optimistic-agent-orchestration/
```

Place that directory where your Agent Skills-compatible client discovers skills. The repository does not require a plugin system, task tracker, or workflow engine.

Example invocation:

```text
Use $optimistic-agent-orchestration to decide which part of this blocked task can start now and what must be revalidated before merge.
```

## Repository layout

```text
.
├── LICENSE
├── README.md
└── skills/
    └── optimistic-agent-orchestration/
        ├── SKILL.md
        └── evals/
            └── cases.md
```

The evaluation cases test behavior and trigger boundaries. They are not a runtime dependency.

## Prior art and scope

This Skill does not claim a new concurrency-control algorithm. It applies established ideas from optimistic concurrency control and speculative execution to Coding Agent workflows:

- H. T. Kung and John T. Robinson, [On Optimistic Methods for Concurrency Control](https://doi.org/10.1145/319566.319567)
- David R. Jefferson, [Virtual Time](https://doi.org/10.1145/3916.3988)
- Cristian Tapus and Jason Hickey, [Distributed speculative execution for reliability and fault tolerance](https://doi.org/10.1007/s00446-008-0073-1)

The mapping to agent work is operational guidance: make assumptions explicit, keep speculative work reversible, validate before authority, and stop adding work when convergence becomes the bottleneck.

## License

Apache License 2.0. See [LICENSE](LICENSE).
