---
name: next-step
description: Drives one step of the build loop. Reads docs/dag.md and docs/state.md to identify what is ready to run, generates a milestone spec and tests for the chosen step, builds, verifies, and issues a PR on completion. Use when the human says "next step", "run step [id]", "what's ready", or "continue the build".
---

# Next Step

Execute one step of the project DAG. This skill moves through five phases. There is one human review gate before building begins, and one at the PR. Do not stop for approval at any other point unless a blocking issue is encountered.

---

## Phase 0: Setup

Determine the feature context for this run.

If the human has named a feature (e.g. "run next step for the auth feature"), use that name. All artifacts for this run go in `features/[feature-name]/docs/`. The PR branch is `feature/[feature-name]`.

If no feature name is given, use `docs/` at the repo root. The PR branch is `feature/[step-id]`.

Create the feature directory structure if it does not exist:
```
features/[feature-name]/
  docs/
    dag.md       ← read from here if feature-scoped
    state.md     ← read from here if feature-scoped
    specs/       ← milestone specs written here
    tests/       ← test files written here
```

---

## Phase 1: Identify

Read `dag.md` and `state.md` from the appropriate path.

Check that all prereqs in `dag.md` have `status: complete`. If any prereq is still `pending`, stop and tell the human which ones need to be completed before building can begin.

Compute which steps are **unblocked**: a step is unblocked when every ID in its `depends_on` list appears in `completed_steps` in state.md, and its own status is `pending`.

If the human named a specific step ID, use that step — confirm it is unblocked before proceeding.

If multiple steps are unblocked, present the options (name, description, what it produces) and ask the human to choose.

If one step is unblocked, name it and ask the human to confirm.

If no steps are unblocked and not all steps are complete, report the blockage and stop.

If all steps are complete, say so and stop.

---

## Phase 2: Specify and Test

Generate the milestone spec and test file for the chosen step together. Present both to the human before building anything.

### Milestone spec

Generate using Deictic Statute format, scoped to this step. Read `initial-spec.md` to stay coherent with project intent. Do not introduce scope beyond what this step's DAG node describes.

```
### Purpose
Why this step exists within the project. One paragraph.
Name the capability being established and why it is necessary for downstream steps.

### Behavior
What must be true when this step is complete.
Constitutive, not descriptive. No implementation language.

### Signal
Specific, verifiable acceptance conditions.
Each condition must be testable without background knowledge.
```

Save to `[path]/specs/[step-id].md`.

### Test file

Derive Gherkin scenarios directly from the Signal section. Each Signal condition becomes one or more scenarios. Scenarios must be specific enough that a developer could implement them without reading the spec.

Save to `[path]/tests/[step-id].feature`.

### Gate

**STOP. Present both files and ask:**
- Does the scope feel right for this step?
- Do the scenarios cover the Signal conditions?
- Anything missing before building begins?

Do not proceed to Phase 3 until the human says approved.

---

## Phase 3: Build

Mark the step as `in_progress` in `state.md` before writing any code.

Implement against the approved spec and tests. Follow all conventions in `CLAUDE.md`.

Run tests when implementation is complete. Fix failures. Do not stop for individual failures — fix and continue.

### Exception handling

At every decision point, ask: **can this be resolved by reasoning, or does it require knowledge only the human has?**

**Handle autonomously — note in PR under Decisions:**
- Implementation choices not specified in the spec
- Bug fixes
- Minor ambiguities where one interpretation is clearly more consistent with spec intent
- Anything covered by CLAUDE.md conventions

**Flag as advisory — note in PR under Issues for Review:**
- Assumption made to continue where spec is ambiguous between two reasonable interpretations
- Non-obvious architectural choice where the wrong call would be expensive to undo
- Continue building with the assumption, document it clearly

**Flag as blocking — create partial PR immediately:**
- Two spec requirements are mutually exclusive and cannot both be satisfied
- A missing prereq is preventing progress (secret, dependency, external service)
- Tests cannot pass because the spec itself contains a conflict
- Do not continue past a blocking issue. Commit what exists, open the PR, stop.

---

## Phase 4: PR

Create a pull request from the feature branch to main.

**PR title:** `[step-id]: [step name]`

**PR description:**

```
## What was built
[One paragraph summary of what this step delivered]

## Tests
[Pass/fail summary. List any tests that were skipped and why.]

## Decisions made autonomously
[Non-obvious choices made during the build that the human should be aware of.
Empty if none.]

## Issues for review
[Assumptions made where the spec was ambiguous. Each entry states:
- what the ambiguity was
- what was assumed
- what to change if the assumption was wrong
Empty if none.]

## Blocking issues
[Only present if the build stopped early. States exactly what is blocking
and what decision or action is needed to unblock.]
```

---

## Phase 5: Complete

If the build completed without blocking issues, update `state.md`:

```yaml
last_updated: "[today's date]"
completed_steps:
  - [all previously completed steps]
  - [this step id]
in_progress: []
decisions:
  - step: "[this step id]"
    note: "[any significant decision worth remembering for future steps]"
notes: ""
```

Report:
- What was completed
- Which steps are now unblocked
- Whether any of those can run concurrently
- Link to the open PR

Stop. The human reviews and merges the PR. The next run of `/next-step` begins after merge.
