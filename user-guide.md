# User Guide

A build framework for Claude Code, distributed as a template repository. The spec for each feature is authored upstream — in a claude.ai chat with MCP repo access — and one command, `/ship`, carries it from committed markdown to a verified deployed release. The spec and its acceptance tests are the contract between what you wanted and what got built.

---

## The division of labor

- **You own intent and the definition of done.** The spec (authored with the upstream chat), the constitution's standards and accepted risks, the answers to `/ship`'s upfront questions, and PR approval.
- **The upstream chat session owns spec authoring**, per `spec-guide.md` — it reads the declaration and constitution first, so specs arrive consistent with the project's accumulated judgment. Picking the next Roadmap item happens in that conversation too.
- **`/ship` owns everything between committed spec and verified release**: the intake checks, the acceptance tests, the adversarial gate, the build, the PR, and the deployment watch.
- **The constitution accumulates the judgment** — lessons, risks, decisions — so no session starts from zero.

Independence is structural: the spec's author (the chat), the tests-and-code author (the `/ship` session), and the reviewer (the clean-context gate subagent) are three separate contexts. None inherits the others' blind spots.

## The loop

1. **Chat** (claude.ai + MCP): talk through the feature; the chat commits `features/<name>-<number>/spec.md` to `main`. For UI-shaped features, Claude Design artifacts are committed alongside and referenced; backend/CLI work skips design entirely.
2. **Claude Code**: `/ship feature-name: <name>`. One upfront question batch, then autonomous. A HIGH adversarial-gate finding is the only mid-run pause.
3. **You**: answer the batch (and any HIGH finding), then review the PR. The body carries the plain-language summary, full-suite test results, deviations, **Risks for review** (merging acknowledges them — they land in the constitution's risk table), and proposed spec-authoring lessons.
4. **The session watches the release**: after merge it polls the Actions deploy, verifies real health (not just green CI), fixes code-level failures via new PRs, raises environmental failures with a diagnosis, and ends with a final report once the release is verified healthy.

## Small changes

No command. Open a session, describe the change, review the PR — the PR body is the documentation. (This replaces the old `/patch`.)

## A fresh project

`/setup` once (operational fields + declaration, one session, one PR) → merge → author the first spec per `spec-guide.md`. The first feature is a **walking skeleton**: the thinnest slice that exercises the Shape's seams end to end, not the most valuable feature — depth comes from feature 2 onward.

---

## Core documents

- **`declaration.md`** — the project's statement of intent, plus **Shape** (3–7 named components/seams) and **Roadmap** (3–7 anticipated features) — both revisable; the Roadmap is what the upstream chat reads as the backlog. Revise it by asking any session (or the chat, via MCP) to edit it deliberately.
- **`constitution.md`** — accumulated judgment: standards registry, architectural principles, patterns, the **Build contract** `/ship` holds every build to, spec-authoring lessons (fed back from build deviations), quality gates, testing wiring, decision log, acknowledged risks.
- **`spec-guide.md`** — the upstream authoring contract: what a spec must contain, where it lands, what it must not contain.
- **`CLAUDE.md`** — the app's operational map: repo layout, run/test/deploy, secrets convention, precedent repos, user-global coordinates.

## Who writes which files

| Writer | Writes / edits |
|---|---|
| Upstream chat | `features/<name>-<number>/spec.md` (+ design artifacts), committed to `main` |
| `/setup` | `README.md`, `CLAUDE.md` project sections, `declaration.md` |
| `/ship` | feature source code, `features/.../tests/`, `build-deviations.md`; **appends** to `spec.md` (`## Coverage`, `## Adversarial gate`) and to `constitution.md` (`## Acknowledged risks`, `## Testing` on first use, `## Spec-authoring lessons`) |
| `/upgrade` | framework-owned files wholesale; `CLAUDE.md`/`constitution.md` framework sections behind per-edit approval |

The rule behind the table: `declaration.md` changes only deliberately (via `/setup` or an explicit edit you direct); `constitution.md` grows by append during `/ship` and changes its principle/pattern/standard sections only deliberately.

## Keeping the framework up to date

`/upgrade` fetches the latest framework-owned files, removes retired command files from earlier framework versions, and merges the framework-owned sections of `CLAUDE.md`/`constitution.md` behind per-edit approval. `FRAMEWORK_VERSION` holds the last-fetched date.
