---
description: Coached conversation that produces features/[feature-name]-[number]/declaration.md, the feature-level statement of intent anchored to the project declaration.
---

Read declaration.md and constitution.md before starting.

If features/[feature-name]-[number]/declaration.md already exists (beyond template placeholders), ask whether the user wants to refine specific sections or rewrite from scratch.

Phase 1 — coached conversation:
Ask the user to articulate this specific feature:
- **What** — what is this feature and what does it do?
- **Why** — why does it exist at this point in the project, and how does it serve the purpose in declaration.md?
- **Success** — what does success look like for this feature specifically?
- **Out of scope** — what is explicitly not part of this feature?

The feature declaration must be consistent with the project declaration. Surface any tension between them before proceeding — that tension is either a feature mis-scoped or a project declaration that needs revising.

Hold the conversation at the level of intent, not implementation. If the user starts describing how it works, redirect — that content belongs in requirements, design, or research.

Continue until the feature declaration is clearly formed. Do not proceed to Phase 2 until the user confirms.

Phase 2 — structured output:
Write features/[feature-name]-[number]/declaration.md with sections labeled `## What`, `## Why`, `## Success`, `## Out of scope`.

Present the written declaration back to the user for review. Revise until it says what they meant.

Commit the completed declaration.md to the current branch (the sandbox provisions this; do not create a new one).

Exit condition: features/[feature-name]-[number]/declaration.md is populated with all four sections, the user has confirmed it, the feature declaration is consistent with the project declaration, and it is committed.
