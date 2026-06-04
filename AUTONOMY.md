# Autonomous build mode

This repo can build a feature end-to-end without a human in the loop, from a
merged spec to a finished build PR. It is **opt-in per feature**, not the default.
Use it for small, high-confidence, low-blast-radius work — and stay in the loop
for the novel and the dangerous (anything touching money, auth, or irreversible
state). The framework already holds the metadata to make that call before you
kick off: the feature declaration's scope, whether it's a walking skeleton, and
the `## Acknowledged risks` table.

## How it runs

1. `/spec` finishes as usual and opens its `spec: <feature>` handoff PR.
2. **You review the spec PR** — this is the one cheap, high-leverage look you keep.
   To hand the build to the agent, add the **`autonomous-build`** label, then merge.
   Merge without the label and nothing fires; run `/build` by hand as before.
3. `autonomous-build.yml` sees the labeled merge, runs `/build` headlessly to a
   green suite, and opens a `build: <feature>` PR.
4. **You review the build PR.** This is now the first time human eyes meet the
   implementation, so the review does more work than it did mid-build — read the
   diff and `build-deviations.md` together.
5. On merge, the deploy runs. If it fails, `deploy-autofix.yml` opens a
   `fix: deploy …` PR for you to review. If it's healthy first try, you're done.

So your touchpoints collapse to: label-and-merge the spec PR, review-and-merge the
build PR, and — only if the deploy breaks — one more fix PR or a blocked issue.

## The escalation contract

The agents never spin and never fake green. When a run hits something it cannot or
must not resolve, it opens a labeled issue and stops:

- **`build-blocked:`** — a missing secret, a kick-back to `/spec` (a requirement is
  wrong, not just hard), or a suite that won't stabilize.
- **`deploy-blocked:`** — a missing/incorrect secret or an access change the agent
  is fenced from making.

In the interactive flow the stop *was* the notification. Async, these issues are
the notification — watch those two labels (forward GitHub notifications to email,
or wire your Fastmail tooling to them if you want a louder signal).

## The fences (do not loosen without thinking)

- The agent may read logs and push code. It may **not** set/rotate secrets, change
  access or permissions, or alter Fly scaling/billing. Those bounce to you by design.
- `deploy-autofix` opens a PR rather than pushing to main, so a redeploy is always
  gated on your merge. That break in the loop is the safety, not an inconvenience.
- `--max-turns` and `timeout-minutes` are the runaway-cost fences. Tune them after
  watching a couple of real runs; start conservative.

## The dependency that makes this trustworthy

Hands-off, **`build-deviations.md` is your only window into what the build did.**
It stops being a side note and becomes the audit trail. Run `/retro` whenever a
build logs a deviation — that's the loop that turns what the build discovered into
fewer questions in the next `/spec`. The autonomy is exactly as trustworthy as that
log is legible.

## Prerequisites (one-time, and yours to do)

These involve credentials and repo admin, so they're set up by you, not the agent:

1. Install the **Claude GitHub app** on the repo (`/install-github-app` from Claude
   Code, or https://github.com/apps/claude) and add **`ANTHROPIC_API_KEY`** to repo
   secrets. Requires repo admin.
2. Confirm **`FLY_API_TOKEN`** is in repo secrets (you already use it for deploy).
3. Create the **`autonomous-build`** label.
4. In `deploy-autofix.yml`, set `workflows: ["…"]` to the exact `name:` of your
   deploy workflow, and uncomment the toolchain setup steps in `autonomous-build.yml`
   to match your stack.
