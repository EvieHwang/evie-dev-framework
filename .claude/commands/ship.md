---
description: End-to-end delivery of one spec'd feature in a single session — intake the committed spec, ask everything unclear once, write the acceptance tests, run the independent adversarial gate, build until every suite is green, open the PR, then watch the merge and deployment through to a verified healthy release.
---

`/ship` takes a feature from committed spec to deployed release in one session. The spec is authored upstream — a claude.ai chat session with MCP repo access, following `spec-guide.md` — and committed to `main` before this session starts. Three separate contexts touch the feature: the chat that wrote the spec, this session that writes the tests and code, and the clean-context gate that attacks both. That independence is deliberate; preserve it.

**Invocation:** `/ship feature-name: [name]`. The spec lives at `features/[feature-name]-[number]/spec.md`. If the user pasted spec content into this session instead, commit it to that path first, then proceed. If there is no spec at all, stop and point the user at `spec-guide.md` — authoring the spec belongs upstream, not here.

## Stage 0 — Intake

Read: `constitution.md` (apply `## Spec-authoring lessons` and `## Build contract`), `declaration.md`, `CLAUDE.md`, the spec, and any design artifacts it references (Claude Design output committed alongside it).

Check the spec against the project's accumulated judgment — the one thing an upstream-authored spec cannot do for itself:

- **Declaration fit.** Does it conflict with `## Out of scope` or stray from the project's What? Note `## Platform` — an Apple platform activates the HIG lens at the gate.
- **Constitution fit.** Which standards apply; which existing patterns it should reuse; whether it compounds an already-acknowledged risk; which quality gates it must clear.
- **Ground truth.** If the design leans on a precedent repo or in-repo reference material named in `CLAUDE.md`, read it. If access is scoped out of this session, that becomes a Stage 1 question — never an assumption.
- **Standards creep.** A thin feature absorbing a heavy cross-cutting standards set (full WCAG sign-off for a one-page admin tool) is a Stage 1 question, not a silent cost.

## Stage 1 — Questions, once

Collect everything unclear into a single upfront batch and raise it via `AskUserQuestion`: ambiguities in the spec, tension with the declaration or constitution, missing decisions, ground truth you could not verify. Batch the questions so the owner is interrupted once, not serially — in a cloud session the question is what notifies them. Zero questions is a valid outcome when the spec is tight.

After this batch, the session runs autonomously until the PR, with exactly one exception: a HIGH gate finding (Stage 3).

## Stage 2 — Tests

Check `constitution.md`'s `## Testing`. If it names runners, use them; if empty, choose the framework that fits the stack, confirm it in the Stage 1 batch (or with a single question now if it only surfaced here), and populate `## Testing` — including, for a polyglot repo, one block per runner and the wiring that lets each runner discover `features/*/tests/` (a `testpaths` entry, a Vitest `include` glob, an import alias). The wiring is a one-time project decision every feature inherits; do not improvise per-feature path hacks.

Derive two categories from the spec:
1. **Behavioral tests** — does it do what the requirements specify?
2. **Seam tests** — do the design's seams hold (timeouts, error taxonomy, component boundaries)? Assert observable behavior, never implementation shape — no constructor-call assertions, private state, or exact internal signatures.

Discipline:
- **Minimum contract surface.** Name only the interface a test genuinely needs to observe behavior, at the most stable surface available. A weak test creates false confidence; a premature test that pins detail just to be writable forecloses valid implementations. Both are defects, and they pull in opposite directions.
- **Tag each test file** `@frozen` (real public contract — satisfied as written) or `@scaffolding` (surface named provisionally — refinable if the behavior holds). Untagged means `@frozen`. Behavioral assertions default to `@frozen`.
- Repo-relative paths resolve from the test file's own location, never an absolute sandbox path.
- If a requirement cannot be tested as written, do not paper over it with a weak test — fix the requirement (a Stage 1-style question if the fix needs owner judgment).

Tests are expected to fail before the code exists; that is the point. Append a `## Coverage` section to `spec.md` mapping each requirement and seam to its test(s).

## Stage 3 — Independent adversarial gate

Spawn a fresh clean-context subagent (Agent tool, `general-purpose`) whose **only** job is to attack the spec and tests — not build, not defend, not rewrite. Give it `constitution.md`, `declaration.md`, the spec, and the test files. It surfaces a finding only when it can name a concrete failure mode; zero findings is a valid outcome. Lenses: **scope drift** (beyond the spec's intent or the project declaration), **integrity** (documents contradict each other; tests assert implementation shape; an `@frozen` tag pinning a non-contract internal, or `@scaffolding` hiding a real behavioral assertion), **coverage** (unspecified or weakly-tested behavior), **security** (named component + location, honoring `Reuses pattern:` markers as HIGH-only surfaces), **standards** (the constitution's registry), **failure modes** (what breaks silently), and — only when `declaration.md`'s Platform names an Apple platform — **HIG-native** (custom controls/gestures/navigation that a standard platform component already solves; default recommendation is to delete the custom path). Each finding: severity (HIGH/MEDIUM/LOW), lens, concrete failure mode, whether it lands in spec or tests. The gate runs once, not in a loop.

**Disposition policy:**
- **HIGH** — the only mid-run pause. Raise via `AskUserQuestion`: fix, acknowledge, or proceed. This is where a wasted build gets prevented.
- **MEDIUM / LOW** — fix when the fix is unambiguous; otherwise carry into the PR body's **Risks for review** section. Merging the PR acknowledges them.
- **Security fixes re-gate.** When a HIGH or MEDIUM *security* finding is fixed, the fix is written by the same author the gate exists to check — so spawn one fresh clean-context subagent scoped to only that finding and diff: does the fix close the failure mode or relocate it? Once per fixed finding, never a loop.
- Every acknowledged finding — whether by an explicit answer or by merge — gets a row in `constitution.md`'s `## Acknowledged risks` (unmitigated severity: an acknowledged HIGH stays HIGH). Rows for merge-acknowledged findings ride in this feature's PR.

Record the gate's findings and dispositions in an `## Adversarial gate` section at the bottom of `spec.md`.

## Stage 4 — Build

The rules are `constitution.md`'s `## Build contract`; in short: tests are the source of truth; `@frozen` is satisfied as written and `@scaffolding` surfaces may be refined if the behavior holds; a requirement that looks actually *wrong* is raised to the owner via `AskUserQuestion`, never silently patched; design is a recommendation; every divergence is logged in `features/[feature-name]-[number]/build-deviations.md`, written to be legible to the next spec author.

Mechanics:
1. Plan and decompose; dispatch independent parts to parallel subagents where the work warrants it.
2. Before implementing a slice, run its tests and confirm they fail for the right reason — a test that passes before its behavior exists is mistagged or wrong; investigate rather than build past it.
3. Iterate until green. Commit as logical units land.
4. Final full-suite run: **every** runner in `## Testing`, all green. A green backend with an unrun frontend suite is not done.

## Stage 5 — PR

Open one PR against `main` (the harness may have auto-created one on first push — update it, don't duplicate). Ready for review; no reviewers or assignees. Body:

- **Feature** — what it does and why, in plain language for someone who hasn't read the spec.
- **Verification** — the full-suite results across all runners, and a pointer to the Coverage map.
- **Deviations** — link `build-deviations.md` if it exists; note any corrected test.
- **Risks for review** — the carried MEDIUM/LOW gate findings, in product terms, with the staged constitution rows. State plainly: *merging acknowledges these.*
- **Spec-authoring lessons (proposed)** — recurring authoring mistakes the deviations expose, phrased as rules the next spec can apply. These ride as `## Spec-authoring lessons` entries in the constitution within this PR when clearly project-level; otherwise list them for the owner.
- **What happens next** — this session stays alive to watch the deployment after merge.

Subscribe to PR activity (`subscribe_pr_activity`). Respond to review comments and CI failures per their content; push fixes to the branch as needed.

## Stage 6 — Watch the release

The PR merging is the owner's approval — do not merge it yourself. Webhook events do not deliver merges, CI success, or deploy completion, so after opening the PR arrange to re-check state on a schedule rather than waiting for a push that will never come.

Once merged:
1. **Follow the deploy.** Poll the GitHub Actions run for the merge commit on `main` until it completes.
2. **Verify health, not just green CI.** `flyctl status` / `flyctl logs` (or the target's equivalent), plus a real probe — the health endpoint or a key user path. A deploy that "succeeded" into a crash loop has not shipped.
3. **On failure: diagnose and fix.** Read the Actions and `flyctl` logs. A code-level cause → fix it on a new branch, open a new PR, and repeat this watch loop after it merges. An environmental cause (secrets, machine config, health checks — and remember the switch signal: two code fixes that should have moved the symptom but didn't means environmental) → raise it to the owner via `AskUserQuestion` with the diagnosis rather than brute-forcing code changes.
4. **On success: close out.** Unsubscribe from PR activity and deliver the final report: what shipped, the verified release state, risks acknowledged, deviations, any spec-authoring lessons not already committed (present them for the owner to apply) — and **where the Roadmap now stands**: which entry this feature completes and which unbuilt entry is next in `declaration.md`'s Roadmap, so the next upstream spec conversation starts oriented.

Exit condition: the feature's suite is green on `main`, the deployment is verified healthy, acknowledged risks and lessons are recorded in the constitution, and the final report is delivered. The session ends here.
