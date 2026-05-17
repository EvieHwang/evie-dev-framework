---
description: Adversarial review of features/[feature-name]-[number]/ — finds problems in requirements and design. Outputs adversarial-review.md as a living artifact with stable finding IDs.
---

You are a reviewer tasked with finding problems. Zero findings means you have not looked hard enough.

Read in order: constitution.md, declaration.md, features/[feature-name]-[number]/declaration.md, features/[feature-name]-[number]/requirements.md, features/[feature-name]-[number]/design.md. If features/[feature-name]-[number]/adversarial-review.md already exists, read it too.

Review through these lenses simultaneously:
- **Integrity** — do the documents contradict each other? Does the design implement the requirements? Do the requirements serve the feature declaration?
- **Coverage** — what behaviors are unspecified? What edge cases are missing?
- **Security** — what attack surfaces does the design expose? Does it comply with constitution.md's security standards?
- **Standards compliance** — does the design respect every applicable item in constitution.md's standards registry (accessibility, API contracts, interface)?
- **Failure modes** — what breaks silently vs. visibly? What recovery is missing?
- **Scope drift** — what has been added beyond the feature declaration? Beyond the project declaration?

If adversarial-review.md exists, this is a re-run:
- For every finding marked `addressed`, verify it was actually addressed by inspecting the latest requirements.md and design.md. If it was, set status to `resolved` and move it to the Resolved section. If it wasn't, set status back to `open` and note why.
- Findings marked `acknowledged` or `deferred` stay as-is unless something has changed that invalidates the acceptance.
- Add new findings discovered in this pass with new IDs.

Write features/[feature-name]-[number]/adversarial-review.md with two sections:

`## Open findings`
For each open finding:
- **ID** — stable identifier (F-001, F-002, …). Never reuse.
- **Severity** — HIGH (blocks DAG generation), MEDIUM (must be addressed or explicitly acknowledged before DAG), LOW (documented but non-blocking).
- **Lens** — which lens surfaced this.
- **Finding** — what is wrong.
- **Recommended action** — which skill to re-run to address it (typically t3-architecture for design/security issues, t3-requirements for coverage/scope issues, or back to t3-feature-declaration for scope drift that needs re-scoping).
- **Status** — `open` / `addressed` (claimed by req/arch but not yet verified) / `acknowledged` (user accepts the risk without fix) / `deferred` (user defers to future work).

`## Resolved findings`
Findings that were addressed and verified. Keep IDs visible for audit trail; full text can be summarized.

Present the open findings to the user. The user decides which to address, acknowledge, or defer. Resolution happens by re-entering the requirements ↔ architecture loop — adversarial does not modify those documents directly.

Commit the completed adversarial-review.md to the current branch (the sandbox provisions this; do not create a new one).

Exit condition: features/[feature-name]-[number]/adversarial-review.md exists, every open finding has severity / lens / recommended action / status, prior addressed findings have been verified, the user has reviewed, and the file is committed.
