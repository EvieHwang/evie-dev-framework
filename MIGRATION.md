# Migration: V2 → V3 (2026-07-02)

*(The V1 → V2 record lives in git history.)*

## The thesis

V2 collapsed V1's process supervision into two pipeline commands plus utilities, but it still authored specs inside Claude Code and still spelled out mechanics the harness now handles natively. Two things changed since:

1. **Spec authoring moved upstream.** The owner talks through a feature in a claude.ai chat with full MCP repo access — optionally with Claude Design for UI-shaped work — and commits the spec to `main` before any Claude Code session starts.
2. **The harness absorbed the choreography.** AskUserQuestion-based stops, PR flow, branch mechanics, orchestration and parallelization, and PR-event watching are now native behavior that no longer needs to be written into every command.

V3 applies V2's own survival test again — *keep what carries owner intent or constraint; cut what supervises the model's process* — and inverts the center of gravity from a command pipeline to **documents plus one gate**.

## The shape

Chat authors `features/<name>-<number>/spec.md` (per the new `spec-guide.md`) → `/ship` runs end to end in one session: intake checks → one upfront question batch → acceptance tests → independent adversarial gate → build to green → PR → after merge, watch the deployment to a verified healthy release → final report → end.

## Kept

- **`constitution.md`** in full, and it absorbs the build contract (tests as source of truth, `@frozen`/`@scaffolding`, immutable requirements, design-as-recommendation, deviation logging) and the walking-skeleton principle as a new `## Build contract` section and an architectural principle.
- **The independent adversarial gate**, as a stage of `/ship`. Independence improved: spec author (chat), test/code author (`/ship`), and reviewer (clean-context subagent) are three separate contexts. The security-fix re-gate rule is kept.
- **Acknowledged-risk propagation** and the **spec-authoring-lessons feedback loop** — now closed by `/ship`'s PR and final report instead of a separate `/retro`.
- **`/setup`** and **`/upgrade`**, both slimmed.

## Collapsed

- `/spec` + `/build` → `/ship`. The spec-handoff PR is gone — it existed only because spec and build ran in separate sessions.
- `/retro` → the tail of `/ship`: the session that watched the build and deploy already holds the deviations, so it proposes the lessons itself.
- `/backlog`, `/feature`, `/declaration` → the upstream chat conversation (which reads the Roadmap via MCP). Declaration edits are now just deliberate edits you direct.
- `/patch` → nothing. A bounded change is what a plain session does by default.

## Deleted

- Command files: `backlog.md`, `build.md`, `declaration.md`, `feature.md`, `patch.md`, `retro.md`, `spec.md`.
- The **Owner-decision stops notify** boilerplate repeated in all eight V2 commands (harness-native).
- All V1-negation language ("no wave loop, no state.md, no convergence counters") — deprogramming for machinery no current model would invent.
- Harness-native environment mechanics from `CLAUDE.md` (MCP-vs-gh, ephemeral container, branch provisioning).

## New

- **`spec-guide.md`** — the upstream authoring contract, written to be read by the chat session via MCP.
- **Stage 6 deployment watch** — `/ship` follows the merge through Actions and the deploy target to *verified health*, not just green CI. Code-level failures are fixed via new PRs in the same session; environmental failures are raised with a diagnosis.

## Owner-decided policies (2026-07-02)

- **Gate findings:** HIGH pauses the run; MEDIUM/LOW are fixed when unambiguous or carried in the PR body — merging acknowledges them.
- **Deploy failure:** diagnose and fix by default; environmental causes are raised, not brute-forced.

## What to watch in practice

- **Spec quality upstream.** The intake checks and the gate are the only nets under a constitution-blind spec. If `/ship`'s question batches grow large, the upstream chat isn't reading `spec-guide.md`/the constitution closely enough.
- **One-session sprawl.** A very large feature may exceed a session. The fix is upstream: split the spec.
- **Merge-as-acknowledgment.** Risks ride the PR body. If PR bodies go unread, risk acknowledgment silently degrades — the constitution's risk table keeps the record honest, but only if the merge decision was a real read.
