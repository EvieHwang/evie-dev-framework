---
name: generate-dag
description: Generates a dependency execution plan (DAG) from a master spec. Use when starting a new project, when scope changes significantly, or when asked to "generate the DAG", "plan the project", or "create the execution plan". Reads initial-spec.md and CLAUDE.md, produces docs/dag.md and initializes docs/state.md.
---

# Generate DAG

Transform the master spec into a structured execution plan — a dependency graph of buildable steps that can be run in sequence or in parallel. You are sequencing work, not solutioning. Do not make implementation decisions that the spec does not require.

## Step 1: Read inputs

Read these files before generating anything:

1. `initial-spec.md` — the master spec (Deictic Statute)
2. `CLAUDE.md` — environment, conventions, constraints, deployment target
3. If the repo has existing code: scan the directory structure and note what already exists. Name this context explicitly in your reasoning — it will affect sequencing.
4. Any reference docs present in the repo — migration docs, API references, architecture diagrams. Note what each one tells you.

## Step 2: Identify prereqs

Before deriving steps, identify everything that must exist in the environment before any build work can begin. This includes:

- Secrets and credentials (API keys, tokens, passwords)
- External services that must be provisioned (databases, queues, third-party APIs)
- Environment variables the app depends on
- Infrastructure that must exist before deployment (runners, registries, hosting targets)
- Any accounts or access that need to be set up manually

These are not build steps — they are human tasks. List them explicitly. The human must complete all prereqs before running `/next-step`.

## Step 3: Derive the steps

Identify the discrete buildable chunks of work implied by the spec. Each step must:

- Produce something testable and demonstrably complete
- Be scoped to one coherent concern (a module, a capability, an integration)
- Be small enough to execute in a single focused session
- Be a vertical slice where possible — end-to-end thin functionality rather than a horizontal layer

A step is too large if it cannot be verified by a specific set of acceptance tests. Split it.

A step is too small if its output only makes sense in combination with other incomplete work. Merge it.

## Step 4: Identify dependencies

For each step, ask: what must exist before this step can begin? A dependency exists when:

- Step B requires an interface, schema, or module that Step A produces
- Step B's tests cannot run without Step A's output
- Deploying or verifying Step B requires Step A to be live

If two steps share no dependencies in either direction and do not write to the same files, they can run concurrently. Note this explicitly.

## Step 5: Write docs/dag.md

Create or overwrite `docs/dag.md` with the following structure:

```yaml
generated: "[today's date]"
spec: "initial-spec.md"

prereqs:
  - id: "prereq-id-kebab-case"
    description: "What is needed and where to get it"
    type: secret | api_key | environment | external_service | other
    status: pending

steps:
  - id: "step-id-kebab-case"
    name: "Human readable name"
    description: "What this step builds or accomplishes — one sentence"
    depends_on: []           # IDs of steps that must complete first. Empty if none.
    produces: "The artifact or capability delivered on completion"
    files_touched:           # Key files this step creates or significantly modifies
      - "path/to/file"
    status: pending          # Do not change this. pending | in_progress | complete
```

After the YAML block, append a Mermaid diagram showing the dependency flow. The diagram is for human orientation. The YAML is authoritative.

## Step 6: Initialize docs/state.md if absent

Check whether `docs/state.md` exists. If it does not, create it:

```yaml
last_updated: "[today's date]"
completed_steps: []
in_progress: []
decisions: []
notes: ""
```

If it already exists, do not overwrite it.

## Step 7: Stop and report

Do not begin implementation. Present a summary to the human:

**Prereqs** — list every item from the prereqs block. Tell the human these must be marked `complete` in `docs/dag.md` before running `/next-step`. They are responsible for completing these manually.

**Steps** — how many were generated, which have no dependencies and can start immediately, which can run concurrently with each other.

**Assumptions** — anything you inferred that the human should verify before approving.

Ask the human to review `docs/dag.md`, confirm the prereqs list is complete, and explicitly approve before any build work begins.
