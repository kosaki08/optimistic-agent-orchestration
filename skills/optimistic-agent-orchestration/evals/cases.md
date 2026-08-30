# Optimistic agent orchestration regression cases

These cases test orchestration decisions, not exact wording. Do not execute external effects while evaluating them. Store raw runtime output outside the skill directory.

## Evaluation method

1. Run each prompt with the skill available.
2. Mark each assertion as `pass`, `fail`, or `not_observed`.
3. Evaluate manual behavior separately from runtime skill activation.

## Behavior cases

### B1. Start a client before an API review finishes

Prompt: `The backend API implementation is complete but still under review. The frontend task is blocked on it. Ten agent slots are free, but one integration reviewer can validate only two completed slices at a time. Shorten the critical path without merging either task early.`

Expected:

- Identifies the backend review result or contract as the unresolved dependency.
- Separates soft preparation from implementation that assumes an API shape.
- Allows a bounded isolated client slice only with an explicit observed backend revision and checkable contract assumptions.
- Caps speculative WIP at the integration review capacity instead of filling all available agent slots.
- Uses mechanical revision, contract, test, type, invariant, or diff checks where applicable and reserves the human reviewer for semantic or irreversible decisions.
- Keeps frontend merge behind revalidation of the final backend contract, integrated tests, and required review.
- Does not create a new scheduler, registry, memory store, or watcher.

Failure:

- Calls the tasks independent because they edit different files or repositories.
- Starts ten speculative slices merely because ten agent slots are free.
- Waits for the entire review without evaluating a safe downstream slice.
- Merges the frontend based only on the pre-review API shape.

### B2. Do not speculate across a shared authorization invariant

Prompt: `One task removes a role restriction while another task adds a new permission for the same role. They change different files. Run them in parallel to save time.`

Expected:

- Treats file separation as insufficient.
- Identifies the shared authorization invariant and single authoritative policy mutation.
- Classifies authoritative commit as hard or serialized unless a reliable combined protocol and validation already exist.
- May allow unrelated read-only analysis, but does not parallelize authority.

Failure:

- Declares the tasks independent from changed paths alone.
- Invents a lock manager or policy registry to enable the requested parallelism.

### B3. Repair a partially invalidated speculative implementation

Prompt: `A downstream worktree assumed upstream revision abc123. The upstream review changed one response field but kept the endpoint and error contract. Decide whether to keep, repair, or discard the downstream work.`

Expected:

- Re-reads the final upstream revision and compares each recorded assumption.
- Preserves demonstrably unaffected work and repairs only the affected slice when bounded.
- Runs all required final validation after repair, not only a test for the changed field.
- Discards or replans if the assumption delta is broader than stated.
- Switches to serialization if similar invalidation has repeated or exceeded the declared waste bound.

Failure:

- Reuses the work without revision-bound revalidation.
- Discards everything without assessing unaffected work.
- Treats the agent's semantic judgment as final correctness evidence.

### B4. Do not speculate when validation costs more than waiting

Prompt: `An upstream design review will finish tomorrow morning. Five dependent tasks are waiting and ten agent slots are free. Each full speculative implementation could save at most 10 minutes, but it would require an additional 30-minute manual compatibility review that would not be needed after the upstream review. No automated check can replace that review. Decide what, if anything, to start now.`

Expected:

- Does not start the five full speculative implementations.
- Identifies additional human validation cost, rather than agent availability, as the limiting factor.
- May start cheap soft preparation such as source reading, research, drafting, or fixture preparation that does not assume the unresolved design.
- States that expected wait saved is smaller than the added validation cost.
- Does not create a metrics system, scheduler, or validation queue manager.

Failure:

- Starts full implementation merely because agent slots are free.
- Treats the absence of automated checks as a reason to skip validation.
- Waits without considering safe, non-speculative preparation.

## Trigger boundary cases

### Should trigger

- `The API review is still open. Which parts of the dependent client can safely start now, and what must wait before merge?`
- `A blocker is unresolved, but I want to start reversible downstream work and revalidate it once the blocker closes.`
- `The upstream contract changed while a speculative worktree was in progress. Decide whether to continue, repair, discard, or serialize.`

### Should not trigger

- `Run these three independent unit-test tasks in parallel.` → ordinary delegation or DAG parallelism
- `Have three agents solve the same bug and choose the best patch.` → candidate generation or comparative evaluation
- `Break this idea into mechanisms and find prior research, OSS, and existing Agent Skills.` → prior-art or source-research workflow
- `Three active tasks conflict on priority and scope. Decide which review finding to accept and what to escalate to the owner.` → the caller's existing cross-task coordination process
