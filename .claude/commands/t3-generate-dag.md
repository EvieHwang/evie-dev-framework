---
description: Generates features/[feature-name]-[number]/dag.md (dependency graph of build tasks organized into parallel waves) and initializes state.md.
---

Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md, features/[feature-name]-[number]/design.md, and features/[feature-name]-[number]/adversarial-review.md.

Verify the inputs are ready before generating:
- requirements.md and design.md should both report stable (the requirements ↔ architecture loop has converged). If either is unstable, stop and tell the user to complete the loop first.
- adversarial-review.md must have no `open` HIGH-severity findings. MEDIUM findings must be marked `acknowledged` or `deferred`. If anything is blocking, stop and tell the user which findings remain.

Generate a dependency graph of all build tasks. For each task:
- **ID** — stable identifier (T-001, T-002, …).
- **Description** — what the task accomplishes.
- **Inputs** — files, components, or prior tasks it depends on.
- **Outputs** — files or behaviors it produces.
- **Dependencies** — IDs of tasks that must complete first.
- **Wave** — the parallel-execution wave (1, 2, 3, …). Tasks in the same wave can run in parallel; later waves wait for earlier ones.
- **Acceptance condition** — what makes this task done. Must be objectively checkable.

Each task must be atomic and completable in a single session.

Write features/[feature-name]-[number]/dag.md with all tasks, grouped by wave.

Initialize features/[feature-name]-[number]/state.md following the format in constitution.md's `## Artifact formats / State file` section, with every task in `pending` status.

Present the DAG and the initial state to the user for review. Revise until the dependency structure and wave grouping are sound.

Commit the completed dag.md and state.md to the current branch (the sandbox provisions this; do not create a new one).

After committing, recommend the user run `/t3-test-coach` next to generate the test suite before invoking `/t3-build`.

Exit condition: features/[feature-name]-[number]/dag.md exists with all tasks organized into waves, features/[feature-name]-[number]/state.md exists with every task in `pending` status, the user has confirmed the DAG, and both files are committed.
