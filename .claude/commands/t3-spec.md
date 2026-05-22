---
description: Orchestrates the full T3 pre-build pipeline — requirements → architecture (loop until stable) → adversarial (loop to address findings) → DAG → tests → PM summary. Runs autonomously after feature declaration is confirmed, stopping only for decisions that require product judgment. Individual steps can still be run manually at any time for refinement or closer involvement.
---

Read features/[feature-name]-[number]/declaration.md before anything else. If it does not exist or contains only template placeholders, stop immediately — run `/t3-feature-declaration` first and confirm the feature declaration before invoking this skill.

Read constitution.md and declaration.md.

## State detection

Check which artifacts exist in features/[feature-name]-[number]/ and resume from the appropriate point. Pipeline stages in order:

1. requirements.md
2. design.md (requirements ↔ architecture loop)
3. adversarial-review.md
4. dag.md + state.md
5. verify.md + tests/
6. spec-summary.md — terminal artifact

If spec-summary.md already exists and is populated, report that the pipeline is complete and exit.

## Orchestration rules

**Proceed autonomously:**
- Writing or rewriting requirements.md and design.md to incorporate architectural feedback or adversarial findings
- Marking adversarial findings `addressed` in adversarial-review.md after the relevant sub-agent addresses them
- Moving to the next stage when the conditions below are met

**Stop and ask the user** — do not proceed autonomously when:
- Adversarial review surfaces a scope drift finding (lens: "Scope drift") that conflicts with the feature declaration
- Architecture or requirements surfaces tension with the project or feature *declaration* (not just with each other)
- A HIGH adversarial finding cannot be resolved by req/arch changes alone and requires an `acknowledged` or `deferred` decision that only the user can make
- The req↔arch loop has not converged after 3 full cycles — surface the specific point of tension

When stopping, present exactly the product or scope question that requires judgment. Do not list technical options. After the user answers, resume from the point where the pipeline stopped.

## Iteration tracking

Track the number of full requirements ↔ architecture cycles completed in this run. If the count reaches 3 without both stability markers appearing, stop and surface the specific conflict that is preventing convergence.

## Stage execution

Each stage runs as a focused sub-agent spawned via the Agent tool. The sub-agent reads the relevant committed artifacts, does its work, and commits its output. The orchestrator does not carry the sub-agent's reasoning — after each sub-agent exits, it reads the committed artifact from disk to determine the next routing decision.

---

### Stage 1 — Requirements

Spawn a sub-agent with this task:

> Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md. If features/[feature-name]-[number]/design.md exists, read it. If features/[feature-name]-[number]/adversarial-review.md exists, read it and address every open finding whose recommended action is t3-requirements — note each addressed finding inline in requirements.md (e.g., *Addresses adversarial F-003.*) and mark it `addressed` in adversarial-review.md.
>
> Produce behavioral requirements in testable terms: user stories with acceptance criteria, edge cases and failure modes, and out-of-scope statements. If this is a revision, note what changed from the prior version and why. At the bottom, state either "Requirements stable — no architectural feedback to incorporate" or list what still needs architectural resolution and why.
>
> Write features/[feature-name]-[number]/requirements.md and commit it.

After the sub-agent exits, read requirements.md. Extract: does the stability marker appear at the bottom?

---

### Stage 2 — Architecture

Spawn a sub-agent with this task:

> Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md. If features/[feature-name]-[number]/design.md exists, read it. If features/[feature-name]-[number]/adversarial-review.md exists, read it and address every open finding whose recommended action is t3-architecture — note each addressed finding inline in design.md (e.g., *Addresses adversarial F-002, F-005.*) and mark it `addressed` in adversarial-review.md.
>
> Produce the architecture: components, contracts, hard technical constraints, seam relationships. For any component or attack surface that genuinely reuses a pattern from constitution.md's pattern registry, mark it explicitly as `Reuses pattern: [name]`. List any requirements that the architecture implies should change. At the bottom, state either "Architecture stable — no requirements changes flagged" or list what should change and why.
>
> Write features/[feature-name]-[number]/design.md and commit it.

After the sub-agent exits, read design.md. Extract: is the stability marker present? Does it flag requirements changes?

**Loop routing:** If either stability marker is absent, increment the iteration counter and return to Stage 1. If both are present, proceed to Stage 3.

---

### Stage 3 — Adversarial review

Spawn a sub-agent with this task:

> You are a reviewer judging whether requirements and design are ready to build. Zero findings is a valid outcome when the spec is sound. Only surface a finding if you can name a concrete failure mode it would cause — generic concerns are not findings.
>
> Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md, features/[feature-name]-[number]/design.md. If adversarial-review.md already exists, run `git log -1 --format=%H -- features/[feature-name]-[number]/requirements.md features/[feature-name]-[number]/design.md` and compare against the SHA recorded in the existing review's header. If neither file has changed since the last review, report "No changes since last adversarial review at [SHA]" and exit without modifying the file.
>
> Review through lenses: integrity, coverage, security (located findings only — name the specific component and spec section; pre-implementation findings that cannot name a file:line must name the spec section precisely), standards compliance per constitution.md, failure modes, scope drift. Apply pattern-reuse scoping: if design.md marks a surface `Reuses pattern: X`, scope security and failure-modes lenses for that surface to HIGH severity only.
>
> For each open finding: ID (F-001…, never reuse), severity (HIGH / MEDIUM / LOW), lens, finding, recommended action (t3-architecture or t3-requirements), status (open). LOW findings without a named location are dropped. Verified addressed findings move to resolved. Acknowledged findings get appended to constitution.md's Acknowledged risks table.
>
> Write features/[feature-name]-[number]/adversarial-review.md with a one-line header recording the current commit SHAs of requirements.md and design.md. Commit it.

After the sub-agent exits, read adversarial-review.md. Classify each open finding:

- **Scope-level** (lens is "Scope drift", or finding surfaces declaration tension) → stop and ask the user
- **Technical, HIGH or MEDIUM** (integrity, coverage, security, standards, failure modes) → return to Stage 1 to address them; the req/arch sub-agents will read adversarial-review.md and address the relevant findings
- **Technical, LOW** → non-blocking; proceed to Stage 4
- **No open findings** → proceed to Stage 4

---

### Stage 4 — DAG generation

Spawn a sub-agent with this task:

> Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md, features/[feature-name]-[number]/design.md, and features/[feature-name]-[number]/adversarial-review.md.
>
> Verify prerequisites: requirements.md and design.md both carry their stability markers; adversarial-review.md has no open HIGH findings and no open MEDIUM findings. If any condition is unmet, report which one and exit without writing.
>
> Generate a dependency graph of all build tasks. For each task: ID (T-001…), description, inputs, outputs, dependencies, wave, acceptance condition (objectively checkable). Group into parallel waves. Each task must be atomic and completable in a single session with margin — if a task aggregates multiple distinct concerns, split it.
>
> Size check before committing: 1–2 tasks total likely means this is T2-sized (surface this). DAG that does not fit one screen, more than ~3–4 waves, or that introduces a new framework, dependency, or deploy path is too large (surface this and recommend splitting the feature). Walking-skeleton features get accommodation on breadth but not on depth.
>
> Write features/[feature-name]-[number]/dag.md. Initialize features/[feature-name]-[number]/state.md with every task in pending status. Commit both.

After the sub-agent exits, read dag.md. If the sub-agent surfaced a T2 sizing problem or a too-large warning, stop and present this to the user before proceeding to Stage 5.

---

### Stage 5 — Test generation

Spawn a sub-agent with this task:

> Read constitution.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md, features/[feature-name]-[number]/design.md, features/[feature-name]-[number]/dag.md.
>
> Check constitution.md's Testing section. If it names a framework, use it. If empty, choose the framework that best fits the project's stack and any existing tests — write the choice and run command into constitution.md's Testing section before generating tests.
>
> Generate two categories of tests: behavioral (from requirements, verifying what was specified) and structural (from design, verifying the architectural constraints). Tag each test with the DAG task IDs it verifies. Every task in dag.md must have at least one test covering it.
>
> If a requirement cannot be tested as written, surface the specific untestable requirement and exit without writing any test files — a weak test is worse than no test.
>
> Write features/[feature-name]-[number]/verify.md (human-readable coverage summary mapping each DAG task to the tests that verify it) and features/[feature-name]-[number]/tests/ (executable test files). Commit all files, including constitution.md if the Testing section was updated.

After the sub-agent exits, check whether the sub-agent surfaced any untestable requirements. If so, stop and present the gap to the user — the resolution is to update requirements.md (re-entering the loop from Stage 1) before proceeding.

---

### Stage 6 — PM summary

Write features/[feature-name]-[number]/spec-summary.md directly. By this point the orchestrator has read all the relevant committed artifacts and does not need a sub-agent.

**Feature** — What it is and why it exists now. One paragraph from the feature declaration, written for a reader who hasn't seen the spec.

**What it does** — Plain-language description of the core behaviors delivered. Derived from requirements but written for a product manager: no implementation detail, no technical jargon. Describes what a user or operator will experience.

**Risks carried** — Any adversarial findings marked `acknowledged` or `deferred`, stated in product terms: what could go wrong and what decision was made about it. If none, state "No risks acknowledged."

**Out of scope** — What this feature explicitly does not address, drawn from the feature declaration and requirements.

**Build preview** — Number of waves, number of tasks, and the DAG's session-budget assessment. One sentence on whether the DAG fits comfortably in one build session or whether anything about it warrants attention.

**Next step** — "Start a new session and run `/t3-build feature-name: [name]`."

Commit spec-summary.md.

---

## Exit

Present spec-summary.md to the user. The pre-build pipeline is complete.
