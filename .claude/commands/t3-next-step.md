---
description: Executes one wave of features/[feature-name]-[number]/dag.md — implements each task in the wave, runs the tests tagged to those tasks, updates state.md.
---

Read constitution.md, features/[feature-name]-[number]/dag.md, features/[feature-name]-[number]/state.md, and features/[feature-name]-[number]/verify.md. Note the test run command from constitution.md's `## Testing` section.

Identify the next wave to execute:
- If all tasks in state.md are `complete`, exit with "all tasks complete — ready for final test run and PR."
- If any task is `in-progress` or `failed`, do not start a new wave; report the in-flight state and stop.
- Otherwise, the next wave is the lowest-numbered wave with `pending` tasks.

For each task in the identified wave:
- Read only the inputs that task requires (per its DAG entry).
- Set status to `in-progress` in state.md, commit.
- Implement the task.
- Verify against the task's acceptance condition.
- On success: set status to `complete`, note the commit SHA in state.md, commit.
- On failure: set status to `failed`, note the reason in state.md, commit, stop.

Tasks within a wave may be run in parallel via sub-agents (Agent tool). Dependencies are guaranteed satisfied within a wave, so parallel execution is safe.

After all wave tasks complete:
- Run the tests tagged with this wave's task IDs (consult verify.md for the task → test mapping).
- On test failure for a completed task: set the responsible task back to `failed`, note the reason, commit, stop.

Do not advance past one wave. t3-build re-invokes this skill in a loop to drive the full DAG to completion; users invoke it directly to resume a partial DAG (after a session ended) or for manual single-wave execution. One wave per invocation matches the cloud-sandbox session boundary, so state.md is always durable at a clean boundary.

Exit condition: every task in the targeted wave is `complete`, tests tagged to those tasks pass, state.md reflects the current truth, and all changes are committed.
