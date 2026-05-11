Read spec/requirements.md, spec/design.md, and 
docs/adversarial-review.md before starting.

Generate a dependency graph of all build tasks:
- Each task must be atomic and completable in a single session
- Identify dependencies between tasks explicitly
- Group independent tasks into parallel waves
- Each task includes: description, inputs, outputs, 
  dependencies, and acceptance condition

Initialize docs/state.md with all tasks in pending status.

Output: features/[feature-name]-[number]/dag.md
        features/[feature-name]-[number]/state.md

Initialize state.md with all tasks in pending status following the format defined in constitution.md.
