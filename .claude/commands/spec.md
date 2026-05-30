---
description: Orchestrates the full pre-build pipeline — declaration → requirements → architecture (loop until stable) → adversarial (loop to address findings) → DAG → tests → PM summary → PR. Produces the feature declaration inline from the preceding conversation (no separate /feature run required), then runs autonomously, stopping only for decisions that require product judgment. Individual steps can still be run manually at any time for refinement or closer involvement.
---

Read constitution.md and declaration.md.

## State detection

Check which artifacts exist in features/[feature-name]-[number]/ and resume from the appropriate point. Pipeline stages in order:

0. declaration.md (feature-level)
1. requirements.md
2. design.md (requirements ↔ architecture loop)
3. adversarial-review.md
4. verify.md + tests/
5. dag.md + state.md
6. spec-summary.md
7. push + PR — terminal step

If features/[feature-name]-[number]/declaration.md is missing or contains only template placeholders, start at Stage 0. If it exists and is populated, skip to the first incomplete subsequent stage.

If a PR for this branch already exists and spec-summary.md is populated, report that the pipeline is complete and exit.

## How stages run

Each Stage 1–5 runs as a focused sub-agent spawned via the Agent tool. The sub-agent **runs the corresponding skill's own procedure** (`/requirements`, `/architecture`, etc.) — that skill file is the single source of truth for what the stage does, including which files to read, how to handle open adversarial findings, and stability-marker rules. Do not restate the skill's procedure in the spawn prompt.

Two orchestration overrides apply to every spawned sub-agent, stated once here rather than per stage:
- **Autonomous mode.** Do not pause to present output to the user or wait for review — the orchestrator owns all user stops (see Orchestration rules). Commit the artifact and exit.
- **Do not push.** Sub-agents commit only. The orchestrator pushes after each stage.

After a sub-agent exits, the orchestrator reads the committed artifact from disk to make its routing decision, then pushes the branch.

## Orchestration rules

**Proceed autonomously:**
- Writing or rewriting requirements.md and design.md to incorporate architectural feedback or adversarial findings
- Marking adversarial findings `addressed` in adversarial-review.md after the relevant sub-agent addresses them
- Moving to the next stage when the conditions below are met
- Pushing the branch (`git push -u origin <current-branch>`) after each stage's commit lands

**Stop and ask the user** — do not proceed autonomously when:
- Adversarial review surfaces a scope drift finding (lens: "Scope drift") that conflicts with the feature declaration
- Architecture or requirements surfaces tension with the project or feature *declaration* (not just with each other)
- A HIGH adversarial finding cannot be resolved by req/arch changes alone and requires an `acknowledged` or `deferred` decision that only the user can make
- The req↔arch loop has not converged after 3 cycles (see Iteration tracking) — surface the specific point of tension
- Adversarial findings have bounced back to Stage 1 three times without converging — surface the unresolved findings

When stopping, present exactly the product or scope question that requires judgment. Do not list technical options. After the user answers, resume from the point where the pipeline stopped.

**Async sub-agent launches.** If the Agent tool returns "Async agent launched" (harness behavior on long-running agents), wait for the completion notification before launching the next stage. Do not poll or fire additional tool calls in the meantime.

## Iteration tracking

Two independent counters, both capped at 3:

- **req↔arch cycles** — one cycle = one Stage 1 → Stage 2 round-trip. Adversarial findings bouncing back to Stage 1 do not increment this counter.
- **adversarial revision rounds** — one round = one Stage 3 → Stage 1 → Stage 2 → Stage 3 loop triggered by open HIGH/MEDIUM findings.

If either counter reaches 3 without convergence, stop and surface the specific conflict.

## Stage execution

Each stage spawns a sub-agent that runs the named skill's procedure under the two orchestration overrides above (autonomous mode, do not push). The spawn prompt names only the skill and any routing detail the orchestrator needs back; it does not restate the skill.

---

### Stage 0 — Declaration

Skip this stage if features/[feature-name]-[number]/declaration.md already exists and is populated (a prior `/feature` run, or a resumed `/spec` session).

Otherwise, produce the feature declaration inline. The orchestrator runs this stage itself — no sub-agent — because the input is the preceding conversation, which only the orchestrator has.

**First-feature mode (walking skeleton).** If no prior feature folders exist under `features/`, this is the project's first feature. Run the coached walking-skeleton path from `/feature` (deliver the prelude unless the preceding conversation has already articulated a thin cross-cutting slice or a Roadmap entry explicitly labeled walking-skeleton/skeleton/spine). The Success section should describe end-to-end reachability across declaration.md's Shape seams, not depth on any one seam.

**Normal mode (feature 2+).** Look at declaration.md's Roadmap and identify the next unbuilt entry. The seed for the declaration is the preceding conversation in this session — what the user has just articulated as the work they want to begin. If that conversation is empty or too thin to extract intent from, prompt the user for a one-paragraph statement of what this feature is and why now, then proceed.

In both modes, the declaration must contain five sections: **What**, **Why**, **Success**, **Shape touched** (list explicit components from declaration.md's Shape), **Out of scope**. Hold it at the level of intent, not implementation.

**Roadmap alignment.** If the proposed feature does not match the next Roadmap entry, surface the divergence in one sentence and ask whether the Roadmap should be updated to reflect the new sequence. Divergence is fine but should be made explicit. If declaration.md has no Roadmap section, recommend running `/declaration` to add one before the next feature; proceed for now.

**Size sanity-check.** Before writing the file:
- Can the value be described in one sentence without "and" doing heavy lifting? Multiple "ands" usually means multiple features.
- First-feature mode: shallow across many seams is correct.
- Normal mode: deep on a coherent slice is correct; broad and shallow across many seams (outside the skeleton case) usually means the feature should be split.

If the size signals fail, stop and surface the mis-sizing to the user before writing. Re-doing the declaration is much cheaper than re-doing requirements, design, and DAG.

**Declaration ↔ project consistency.** Surface any tension between the feature declaration and declaration.md before writing. If a feature requires a component not listed in the Shape, surface that too — either the Shape was incomplete (update declaration.md) or the feature is straying out of scope.

Write features/[feature-name]-[number]/declaration.md with sections labeled `## What`, `## Why`, `## Success`, `## Shape touched`, `## Out of scope`. If this is a walking skeleton, note that explicitly at the top of the file (e.g., `*Walking skeleton — first feature.*`).

Present the written declaration back to the user for confirmation. Revise until it says what they meant. Do not proceed to Stage 1 until the user confirms.

Commit declaration.md and push the branch. **Do not open a PR at this stage** — the spec PR is opened at Stage 7. The cloud harness may auto-create one on first push; if so, leave it as-is (Stage 7 will update it).

---

### Stage 1 — Requirements

Spawn a sub-agent: **run the `/requirements` procedure.**

After it exits, read requirements.md: does the stability marker appear at the bottom? Push the branch.

---

### Stage 2 — Architecture

Spawn a sub-agent: **run the `/architecture` procedure.**

After it exits, read design.md and requirements.md: are both stability markers present? Does design flag requirements changes? Push the branch.

**Loop routing:** If either stability marker is absent (and the gap is not marker-only — design didn't flip it because text actually needs changing), increment the req↔arch counter and return to Stage 1. If both are present, proceed to Stage 3.

---

### Stage 3 — Adversarial review

Pick the mode the `/adversarial` skill defines (fresh review vs. verification pass) based on the diff since the last adversarial-review.md, and name it in the spawn prompt.

Spawn a sub-agent: **run the `/adversarial` procedure in [fresh review | verification pass] mode.**

After it exits, read adversarial-review.md, then push the branch. Classify each open finding:

- **Prescription feedback** (`## Prescription feedback` section in adversarial-review.md is non-empty) → return to Stage 2 without incrementing the adversarial counter. The architecture sub-agent will read the feedback and restate prescriptions as behavioral constraints. Do not also return to Stage 1 unless requirements changes are independently needed.
- **Scope-level** (lens is "Scope drift", or finding surfaces declaration tension) → stop and ask the user
- **Technical, HIGH or MEDIUM** → increment the adversarial-revision counter and return to Stage 1; the req/arch sub-agents will read adversarial-review.md and address the relevant findings
- **Technical, LOW** → non-blocking; proceed to Stage 4
- **No open findings and no prescription feedback** → proceed to Stage 4

---

### Stage 4 — Test generation

Spawn a sub-agent: **run the `/tests` procedure.**

After it exits, push the branch. If the sub-agent surfaced an untestable requirement, stop and present the gap to the user — the resolution is to update requirements.md (re-entering the loop from Stage 1) before proceeding to Stage 5.

---

### Stage 5 — DAG generation

Spawn a sub-agent: **run the `/dag` procedure.**

After it exits, read dag.md and verify.md and push the branch. If the sub-agent surfaced a sizing problem, a too-large warning, or a task with no test coverage, stop and present this to the user before proceeding to Stage 6.

---

### Stage 6 — PM summary

Write features/[feature-name]-[number]/spec-summary.md directly. By this point the orchestrator has read all the relevant committed artifacts and does not need a sub-agent.

**Feature** — What it is and why it exists now. One paragraph from the feature declaration, written for a reader who hasn't seen the spec.

**What it does** — Plain-language description of the core behaviors delivered. Derived from requirements but written for a product manager: no implementation detail, no technical jargon. Describes what a user or operator will experience.

**Risks carried** — Any adversarial findings marked `acknowledged` or `deferred`, stated in product terms: what could go wrong and what decision was made about it. If none, state "No risks acknowledged."

**Out of scope** — What this feature explicitly does not address, drawn from the feature declaration and requirements.

**Build preview** — Number of waves, number of tasks, and the DAG's session-budget assessment. One sentence on whether the DAG fits comfortably in one build session or whether anything about it warrants attention.

**Next step** — "Review and merge the spec PR, then start a new session and run `/build feature-name: [name]`. The build starts fresh from `main`, so the spec must be merged first."

Commit spec-summary.md and push the branch.

---

### Stage 7 — Ensure the handoff PR

**Session model.** `/spec` typically runs in one session from declaration through DAG. `/build`, by contrast, *always* runs in a separate session: a re-cloned sandbox provisions a new branch off `main`. That means the spec artifacts must be merged to `main` before `/build` can begin. This PR is that merge gate.

The cloud sandbox is ephemeral — the spec is only handed off to the next session via this PR.

Push the branch one final time to ensure all commits are upstream. Then ensure exactly one handoff PR exists for this branch against `main` (use the GitHub MCP server). Check for an existing PR first (`mcp__github__list_pull_requests` for the branch):

- **If a PR already exists** (either auto-created by the harness on first push, or opened by an earlier `/feature` run): update it (`mcp__github__update_pull_request`) to serve as the spec handoff. Do not open a second PR.
- **If none exists:** open one (`mcp__github__create_pull_request`).

Either way, the PR should end up as:

- Title: `spec: [feature-name]`
- Body: include spec-summary.md content plus a one-line note that the PR is the handoff to the next session for `/build`, and that it must be merged to `main` before `/build` starts.
- Ready for review (not draft) — flip it from draft if it was opened as one.
- No reviewers or assignees, per CLAUDE.md (the owner is the PR author).

Do not merge the PR. Present the PR URL to the user and tell them to review and merge it, then start a new session and run `/build` from `main`. The pre-build pipeline is complete.
