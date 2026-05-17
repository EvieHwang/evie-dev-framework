---
description: Coached conversation that populates the project-level declaration.md (what / why / for whom / out of scope).
---

Read the current declaration.md before starting.

Phase 1 — coached conversation:
Ask about:
- **What** — what this project is, in one or two sentences with no implementation detail.
- **Why** — the problem it solves or the need it serves.
- **For whom** — who uses it and what they need from it.
- **Out of scope** — what this project explicitly does not do.

If declaration.md is already populated (beyond template placeholders), ask whether the user wants to refine specific sections or rewrite from scratch.

Hold the conversation at the level of intent, not implementation. If the user starts describing how it works rather than what it is, redirect — that content belongs in constitution.md or feature artifacts.

Continue until the declaration is clearly formed. Do not proceed to Phase 2 until the user confirms.

Phase 2 — structured output:
Write declaration.md (at the repo root) with sections labeled `## What`, `## Why`, `## For whom`, `## Out of scope`.

Present the written declaration back to the user for review. Revise until it says what they meant.

Commit the completed declaration.md to the current branch (the sandbox provisions this; do not create a new one).

Exit condition: declaration.md is populated with all four sections, the user has confirmed it captures their intent, and it is committed.
