---
description: Upgrade a downstream project's framework-owned files to the latest version from evie-dev-framework. Replaces the commands, spec-guide.md, user-guide.md, features/README.md, and FRAMEWORK_VERSION wholesale; semantically merges CLAUDE.md and constitution.md framework-owned sections behind per-edit in-session approval, preserving project-specific content.
---

Upgrade this project's framework files to the latest version from the canonical framework repo.

## Source of truth

Declared once and used for **every** fetch below — never inline another repo slug:

```
SOURCE_REPO = EvieHwang/evie-dev-framework
```

Fetch with `curl` against raw GitHub URLs (`https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/<path>`). Do not use `mcp__github__get_file_contents` for the framework repo (MCP is scoped to the current project) and do not use WebFetch.

## Pre-flight checks

Stop and report if any check fails.

1. **Uncommitted changes.** `git status --porcelain` must be empty — otherwise tell the user to commit or stash first.
2. **Project is set up.** If `CLAUDE.md` still contains `# [Project Name]`, stop — run `/setup` first.
3. **Versions.** Local `FRAMEWORK_VERSION` is `from_version` (missing file → `(pre-versioning)`). `curl -sf .../FRAMEWORK_VERSION`, stripped, is `to_version`; stop on curl failure. If equal, report "Already up to date at framework version `<to_version>`." and exit.

## Replace framework-owned files

For each path below: `curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/<path> -o <path>` (`mkdir -p` parents as needed). Verbatim — no merging. Stop and report if any curl fails, before committing anything.

```
FRAMEWORK_VERSION
user-guide.md
spec-guide.md
features/README.md
.claude/commands/ship.md
.claude/commands/setup.md
.claude/commands/upgrade.md
```

**Removing stale commands.** A project upgrading from an earlier framework version may still carry retired command files under `.claude/commands/`: the V1 set (`requirements.md`, `architecture.md`, `adversarial.md`, `tests.md`, `dag.md`, `next.md`) and/or the V2 set (`backlog.md`, `build.md`, `declaration.md`, `feature.md`, `patch.md`, `retro.md`, `spec.md`). `git rm` any that exist — they were collapsed into `/ship` (with spec authoring moved upstream per `spec-guide.md`), and a lingering file would shadow the current flow. Note in the PR which were removed. Also `git rm` a stale `MIGRATION.md` if present — it describes a boundary this upgrade crosses.

## Structural merge — CLAUDE.md and constitution.md

These mix framework-template sections with project-specific content, so they are **never replaced wholesale**. Fetch the framework versions of both once, then build the full list of proposed edits across the two classes below and walk them one at a time through the approval step.

**Class 1 — framework-owned section merges.** Compare framework vs. local for: `CLAUDE.md` → `## Repo map`, `## Development environment`, `## Secrets`; `constitution.md` → `## Standards`, `## Acceptance bar`, `## Spec-authoring lessons`. Byte-identical → skip. Absent locally (newly introduced, e.g. `## Acceptance bar`) → propose adding the framework version verbatim at its framework-file position. Differs → produce a **semantic merge**: adopt the framework's structure and new directives while preserving project-specific lines the local copy added (repo-specific Repo map entries, project-added standards, accumulated lessons). When unsure whether a local line is a project addition or stale framework text, keep it and flag it in the approval prompt.

**Class 2 — stale-section deletions.** Propose deleting, if still present locally: `CLAUDE.md` → `## Build flow note`; `constitution.md` → `## Artifact formats` and `## Build contract` (renamed to `## Acceptance bar`, added by Class 1 — remove the old section so the two don't coexist; a project that added its own bullets to `## Build contract` should have them flagged for manual re-homing, not silently dropped). Pure framework removals otherwise — no project content lives in them.

**Approval — one prompt per edit.** Use `AskUserQuestion` per edit with options **Apply**, **Skip**, and **Show full** (print the complete before/after, then re-ask). Include enough diff in the question to decide without scrolling back. Apply each approved edit immediately with the `Edit` tool; leave skipped edits untouched. Record applied vs. skipped for the PR body. If nothing differs, report "No CLAUDE.md / constitution.md changes to apply."

## Commit, push, PR

Stage the replaced framework files, plus `CLAUDE.md`/`constitution.md` only if at least one approved edit touched them. Never stage other project files.

```
chore: upgrade framework to <to_version>

Updated from <from_version>. Commands, spec-guide.md, user-guide.md,
features/README.md, and FRAMEWORK_VERSION replaced wholesale.
CLAUDE.md / constitution.md framework-owned sections merged with
approval (see PR for applied vs. skipped).
```

Push (`git push -u origin <current-branch>`) and open a PR against `main` — title `chore: upgrade framework to <to_version>`; body listing the upgraded files, removed stale commands, and the applied vs. skipped CLAUDE.md/constitution.md edits (so a skipped one can be revisited manually). Ready for review; no reviewers or assignees. Present the PR URL.
