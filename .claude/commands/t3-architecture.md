---
description: Produces features/[feature-name]-[number]/design.md. Re-run as part of the requirements ↔ architecture loop until convergence.
---

Read constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, and features/[feature-name]-[number]/requirements.md. If features/[feature-name]-[number]/design.md exists, read it too.

Produce the architecture given the current requirements:
- Components, contracts, and hard technical constraints.
- How components relate and where the seams are.
- What the build agent must know before touching code.

Then list any requirements that the architecture implies should change. Do not silently reshape requirements — surface them explicitly.

If the surfaced list is non-empty, the user should run t3-requirements next to incorporate the changes, then re-invoke this skill. The loop continues until neither side flags new changes.

If the surfaced list is empty, this side of the loop is stable. State that explicitly at the end of design.md ("Architecture stable — no requirements changes flagged"). If t3-requirements also reports stable on its next run, the loop has converged.

If the architecture surfaces tension with the project or feature declaration (not just requirements), stop and surface that separately. Declaration tension is more serious than requirements tension — it usually means the feature is mis-scoped.

Write features/[feature-name]-[number]/design.md.

Present the written design to the user for review.

Commit the completed design.md to the current branch (the sandbox provisions this; do not create a new one).

Exit condition: features/[feature-name]-[number]/design.md exists, the user has confirmed it, any requirement-change flags are explicit, and the file is committed.
