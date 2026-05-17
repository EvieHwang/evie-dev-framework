---
description: Produces features/[feature-name]-[number]/requirements.md in testable terms. v1 hypothesis on first run; v2 committed pass when design.md exists.
---

Read constitution.md, declaration.md, and features/[feature-name]-[number]/declaration.md before starting.

Determine which pass this is:
- If features/[feature-name]-[number]/design.md does not exist, this is the **v1 hypothesis pass**.
- If features/[feature-name]-[number]/design.md exists, this is the **v2 committed pass** — also read design.md and revise requirements to incorporate the architectural constraints it surfaces.

Produce behavioral requirements in testable terms:
- **User stories with acceptance criteria** — each story names a user, a need, and the criteria that confirm it works.
- **Edge cases and failure modes** — what happens at the boundaries and when things go wrong.
- **Out of scope** — what this feature explicitly does not address.

For v1: flag any requirement that architectural review is likely to reshape. v1 is a hypothesis, not a commitment.

For v2: do not flag — requirements should be committed at this point. If architecture surfaced something that genuinely needs more debate, stop and surface it; don't paper over it with a flag.

Write features/[feature-name]-[number]/requirements.md.

Present the written requirements to the user for review. Revise until they are sound.

Commit the completed requirements.md to the current branch (the sandbox provisions this; do not create a new one).

Exit condition: features/[feature-name]-[number]/requirements.md exists, the user has confirmed it, requirements are testable, and the file is committed. For v2 only: requirements reflect the architectural constraints in design.md.
