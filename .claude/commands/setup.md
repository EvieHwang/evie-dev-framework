---
description: Coached repository setup after cloning the template — fills CLAUDE.md (project name, one-liner, run/test/deps, deployment target) and declaration.md in one session, commits both, and opens the setup PR. Run once on a fresh clone before any feature work.
---

Runs once per freshly-cloned template. Read `CLAUDE.md`, `README.md`, and `declaration.md` first. If `CLAUDE.md` no longer contains `# [Project Name]` (the placeholder), the repo has already been set up — confirm with the user before re-running. Do not touch `constitution.md`; its app-specific sections develop as the project does.

## Phase 1 — Operational setup

Four low-stakes fields, usually pre-decided — **default to expert mode**: propose a complete draft of all four at once, inferred from the repo and the conversation, and ask the user to confirm or hand back diffs. Walk one-per-turn only if the user is undecided or asks to be coached.

1. **Project name** — replaces `# [Project Name]` in CLAUDE.md; heads the new README.
2. **One-line description** — README subtitle and CLAUDE.md's description line.
3. **Stack / runtime** — the run/test/deps block (Node + pnpm/npm, Python + pyproject, Swift/Xcode, static site, or "skip — fill later").
4. **Deployment target** — Fly.io (primary), AWS, Apple/Xcode, or "none yet". Pull the canonical coordinates from CLAUDE.md's `## User globals`.

Write and commit together:
- `README.md` — overwrite with the name and one-liner (no template framework text remains).
- `CLAUDE.md` — the name, the one-liner, one uncommented run/test/deps block, one uncommented deployment-target block. Preserve everything else verbatim, especially `## User globals`.

## Phase 2 — Declaration

Seed **What** from the one-liner and cover, at the level of intent (redirect implementation talk):

1. **What** — one or two sentences.
2. **Why** — the problem it solves.
3. **For whom** — who uses it and what they need.
4. **Out of scope** — what it explicitly does not do.
5. **Platform** — iOS / macOS / iPadOS / visionOS / watchOS / tvOS / web / CLI / server / cross-platform. Tell the user: an Apple platform activates the HIG-native lens in `/ship`'s adversarial gate, which pushes back on custom-built controls the platform already provides.
6. **Shape** — 3–7 named components/seams, one line each. A revisable map of the parts, not an architecture; it exists so the first feature has seams to slice against.
7. **Roadmap** — 3–7 anticipated features in rough order, each tagged with the seams it touches. Memory, not commitment — it's what the upstream spec conversation reads to pick the next item.

Shape and Roadmap are the genuine forks — take them one at a time; the rest at whatever cadence the conversation supports. Write `declaration.md` with sections `## What`, `## Why`, `## For whom`, `## Out of scope`, `## Platform`, `## Shape (revisable)`, `## Roadmap (revisable)`. Present it for review; revise until it says what the user meant. Commit.

## Exit

Push the branch (`git push -u origin <current-branch>`) and ensure exactly one setup PR is open against `main` (the harness may have auto-created one on first push — update it rather than opening a second; no reviewers or assignees). Tell the user: merge it, then author the first spec per `spec-guide.md` (a walking skeleton) and run `/ship` in a fresh session.
