# Features

Artifacts for each feature live in their own folder.

**Naming convention:** `[feature-name]-[number]`
- Numbers are sequential per feature name, used for both ordering and disambiguation.
- Do not overwrite a previous version — create a new numbered folder.

Contents:
- `spec.md` — authored **upstream** (a claude.ai chat following `spec-guide.md`) and committed to `main` before `/ship` runs. The spec is the feature's intent and definition of done.
- design artifacts (optional) — Claude Design output the spec references, committed alongside it.
- `tests/` — the executable acceptance suite, written by `/ship`. Once green (and the rest of `constitution.md`'s `## Acceptance bar` is met), the feature is built.

`/ship` reports where the build diverged from the spec's design, and the spec-authoring lessons those divergences imply, in the **PR** — not in per-feature bookkeeping files. Clearly project-level lessons land in `constitution.md`'s `## Spec-authoring lessons`, which the next spec's author reads.

Do not pre-create empty placeholder files.
