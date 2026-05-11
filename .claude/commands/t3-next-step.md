Read docs/dag.md and docs/state.md before starting.

Identify the next ready wave — tasks with all 
dependencies complete.

For each task in the wave:
- Read only the inputs that task requires
- Implement the task
- Verify against the task's acceptance condition
- Update docs/state.md on completion following the format defined in constitution.md.

Run the test suite on completion of each wave.
On test failure: stop and report with full context.
Do not advance to the next wave until all tests pass.
