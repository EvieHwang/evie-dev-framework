# Features

Artifacts for each feature live in their own folder.

**Naming convention:** `[feature-name]-[number]`
- Numbers are sequential per feature name, used for both ordering and disambiguation.
- Do not overwrite a previous version — create a new numbered folder.

Contents:
- `spec.md` — authored **upstream** (a claude.ai chat following `spec-guide.md`) and committed to `main` before `/ship` runs. `/ship` appends the `## Coverage` map and the `## Adversarial gate` record.
- design artifacts (optional) — Claude Design output the spec references, committed alongside it.
- `tests/` — the executable acceptance suite, written by `/ship`. Once green, the feature is built.
- `build-deviations.md` — created by `/ship` only if implementation diverged from the design or a test was corrected. It is the feedback channel back to authoring: `/ship`'s closing report mines it for recurring spec-authoring mistakes and routes them to `constitution.md`'s `## Spec-authoring lessons`, which the next spec's author reads.

Do not pre-create empty placeholder files.
