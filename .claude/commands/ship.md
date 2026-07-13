---
description: Take one committed spec to a verified deployed release in a single session — check it against the project's accumulated judgment, ask everything unclear once, build it to the constitution's acceptance bar, get an independent risk review, open the PR, then watch the merge and deployment through to a verified healthy release.
---

`/ship` takes a feature from committed spec to deployed release in one session. The spec is authored upstream — a claude.ai chat with MCP repo access, following `spec-guide.md` — and committed to `main` before this session starts.

The framework specifies the **edges and the outputs**, not the path between them. What to build comes in as the spec; what "done" means is `constitution.md`'s `## Acceptance bar`; how the build gets there is yours. Don't expect a step-by-step procedure below — expect a definition of done and the few checkpoints that protect it.

**Invocation:** `/ship feature-name: [name]`. The spec lives at `features/[feature-name]-[number]/spec.md`. If the user pasted spec content into this session instead, commit it to that path first, then proceed. If there is no spec at all, stop and point the user at `spec-guide.md` — authoring the spec belongs upstream, not here.

## Intake

Read `constitution.md` (the `## Acceptance bar` is your definition of done; apply `## Spec-authoring lessons`, `## Patterns in use`, and the acknowledged risks), `declaration.md`, `CLAUDE.md`, the spec, and any design artifacts it references. Then check the spec against the project's accumulated judgment — the one thing an upstream-authored spec cannot do for itself:

- **Declaration fit** — does it conflict with `## Out of scope` or stray from the What? Note `## Platform`; an Apple platform makes native-fit part of the risk review.
- **Constitution fit** — which standards actually apply (proportionate to what it touches), which existing patterns it should reuse, whether it compounds an already-acknowledged risk.
- **Ground truth** — if the design leans on a precedent repo or in-repo reference named in `CLAUDE.md`, read it. If access is scoped out of this session, that's a question below, not an assumption.

## Questions, once

Collect everything unclear into a single upfront batch and raise it via `AskUserQuestion`: spec ambiguities, tension with the declaration or constitution, missing decisions, ground truth you couldn't verify. In a cloud session the question is what notifies the owner, so interrupt them once, not serially. Zero questions is a valid outcome when the spec is tight. After this batch the session runs autonomously to the PR, with one exception: a HIGH risk-review finding.

## Build to the acceptance bar

Build the feature until it clears `constitution.md`'s `## Acceptance bar`. That bar is the contract; the route to it is your call. In practice that means: the behavioral requirements are satisfied and covered by green tests (every runner in `## Testing`), the quality gates pass, applicable standards are honored, and spec integrity is preserved — a requirement that looks *wrong* is raised to the owner via `AskUserQuestion`, never silently patched.

- On first use, populate `constitution.md`'s `## Testing` — the runner(s) for the stack and the one-time wiring that lets each discover `features/*/tests/`. This is a project decision every feature inherits, not a per-feature path hack.
- Decompose and parallelize where the work warrants it; commit as logical units land.
- Where the build had to diverge from the spec's *design* to satisfy the behavior, note it for the PR (and propose the spec-authoring lesson it implies). No prescribed file or format — the PR is the channel.
- Final run: **every** runner green. A green backend with an unrun frontend suite is not done.

## Independent risk review

Before opening the PR, get an independent read on what was built — the one check the builder can't perform on itself, because it would be grading its own homework. Scale it to the feature's actual surface: a feature with external input, auth, a security-relevant seam, or an Apple-platform interface warrants a real adversarial pass; a low-stakes internal change may warrant little or none. Say which you judged it to be.

When a real pass is warranted, the reliable way to get genuine independence today is a fresh clean-context subagent (Agent tool, `general-purpose`) given `constitution.md`, `declaration.md`, the spec, and the tests, whose only job is to attack — not build, not defend. Useful lenses: scope drift, coverage gaps, security (named component + location), standards, silent failure modes, and — on an Apple platform — custom controls a native component already solves. It surfaces a finding only when it can name a concrete failure mode; zero is valid.

Disposition:
- **HIGH** — the only mid-run pause. Raise via `AskUserQuestion`: fix, acknowledge, or proceed. This is where a wasted build gets caught.
- **MEDIUM / LOW** — fix when unambiguous; otherwise carry into the PR's **Risks for review**. Merging acknowledges them.
- A fixed HIGH/MEDIUM **security** finding gets one clean-context re-check scoped to that finding and diff — does the fix close the failure mode or relocate it? Once, not a loop.
- Every acknowledged finding gets a row in `constitution.md`'s `## Acknowledged risks` (unmitigated severity). Rows for merge-acknowledged findings ride in this PR.

## PR

Open one PR against `main` (the harness may have auto-created one on first push — update it, don't duplicate). Ready for review; no reviewers or assignees. The body is the acceptance report:

- **Feature** — what it does and why, in plain language for someone who hasn't read the spec.
- **Verification** — full-suite results across every runner, and how health was (or will be) confirmed.
- **Deviations** — where the build diverged from the spec's design, and any corrected test.
- **Risks for review** — the carried MEDIUM/LOW findings in product terms, with the staged constitution rows. State plainly: *merging acknowledges these.*
- **Spec-authoring lessons (proposed)** — recurring authoring mistakes the deviations expose, phrased as rules the next spec can apply. Commit them as `## Spec-authoring lessons` entries in this PR when clearly project-level; otherwise list them for the owner.

Subscribe to PR activity (`subscribe_pr_activity`); respond to review comments and CI failures on their merits and push fixes as needed.

## Watch the release

The PR merging is the owner's approval — do not merge it yourself. Webhook events don't deliver merges, CI success, or deploy completion, so arrange to re-check state on a schedule rather than waiting for a push that never comes.

Once merged: follow the deploy (poll the Actions run for the merge commit), then **verify health, not just green CI** — `flyctl status` / `flyctl logs` or the target's equivalent, plus a real probe of the health endpoint or a key path. A deploy that "succeeded" into a crash loop has not shipped. On a code-level failure, fix it on a new branch and PR and repeat this watch after it merges. On an environmental failure (secrets, machine config, health checks — the switch signal is two code fixes that should have moved the symptom but didn't), raise it to the owner via `AskUserQuestion` with the diagnosis rather than brute-forcing code.

Exit condition: the feature's suite is green on `main`, the deployment is verified healthy, acknowledged risks and lessons are recorded in the constitution, and the final report is delivered — including **where the Roadmap now stands** (which entry this completes, which unbuilt entry is next) so the next spec conversation starts oriented. The session ends here.
