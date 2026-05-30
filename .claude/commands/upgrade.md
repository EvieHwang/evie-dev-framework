---
description: Upgrade a downstream project's framework-owned files to the latest version from evie-dev-framework. Replaces all skill commands, user-guide.md, features/README.md, and FRAMEWORK_VERSION wholesale; surfaces CLAUDE.md and constitution.md structural changes as a manual review checklist in the PR.
---

Upgrade this project's framework files to the latest version from `EvieHwang/evie-dev-framework`.

All framework files are fetched using `curl` via the Bash tool against raw GitHub URLs (`https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/<path>`). Do not use `mcp__github__get_file_contents` for the framework repo (MCP is scoped to the current project only) and do not use WebFetch (blocked by sandbox network policy).

## Pre-flight checks

Run these before touching any files. Stop and report if any check fails.

1. **Uncommitted changes.** Run `git status --porcelain`. If any output appears, stop — tell the user to commit or stash changes before upgrading.

2. **Project is set up.** Read `CLAUDE.md`. If it still contains `# [Project Name]` (the template placeholder), stop — this project has not been initialized. Run `/setup` first.

3. **Current version.** Read `FRAMEWORK_VERSION` locally. Record as `from_version`. If the file does not exist, `from_version` is `(pre-versioning)`.

4. **Target version.** Run:
   ```bash
   curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/FRAMEWORK_VERSION
   ```
   Strip whitespace from the output. Record as `to_version`. If curl fails (non-zero exit), stop and report the error.

5. **Already current.** If `from_version == to_version`, report "Already up to date at framework version `<to_version>`." and exit without making any changes.

## Replace framework-owned files

For each file in the list below, run:
```bash
curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/<path> -o <path>
```

If the file does not yet exist locally, create any missing parent directories first (`mkdir -p`). Write the fetched content verbatim — do not merge or selectively apply.

```
FRAMEWORK_VERSION
user-guide.md
features/README.md
.claude/commands/build.md
.claude/commands/declaration.md
.claude/commands/feature.md
.claude/commands/patch.md
.claude/commands/retro.md
.claude/commands/setup.md
.claude/commands/spec.md
.claude/commands/upgrade.md
```

If any `curl` call fails, stop and report which file failed before making any commits.

**Removing stale commands.** A project upgrading across the V2 boundary may still have the V1 command files (`requirements.md`, `architecture.md`, `adversarial.md`, `tests.md`, `dag.md`, `next.md`) under `.claude/commands/`. If any exist, `git rm` them — they were collapsed into `/spec` and `/build` and a lingering file would shadow the new flow. Note in the PR which were removed.

## Structural diff — CLAUDE.md and constitution.md

These files mix framework template sections with project-specific content and are **never replaced wholesale**. Instead, compare specific sections and surface differences for manual review in the PR body.

Fetch the framework versions:
```bash
curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/CLAUDE.md
curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/constitution.md
```

**CLAUDE.md** — compare the text of each of these sections (from the heading to the next `##` heading) between the framework version and the local file:
- `## Repo map`
- `## Development environment`
- `## Secrets`

**constitution.md** — compare:
- `## Standards`

For each section that differs, record both versions (labeled `Framework:` and `Local:`) in the PR body under `## Manual review checklist`. If all compared sections are identical, write "No structural changes to review."

**Removed sections.** The structural diff above surfaces sections that *changed*; it does not catch sections the framework has *deleted* but a downstream repo still carries in its always-loaded files. Check for these explicitly, because a lingering one keeps stale rules and references in context:
- `CLAUDE.md` — if it still contains a `## Build flow note` section, flag it for deletion (the DAG / `state.md` / `verify.md` build flow it describes no longer exists).
- `constitution.md` — if it still contains a `## Artifact formats` section (the `state.md` state-file spec), flag it for deletion.

For each removed section still present locally, add an entry under `## Manual review checklist` recommending its deletion and naming what made it obsolete. Do not delete it yourself — these files are never modified by `/upgrade`; the user removes it during manual review.

Do not modify `CLAUDE.md` or `constitution.md`.

## Commit

Stage only the framework-owned files listed above. Do not stage `CLAUDE.md`, `constitution.md`, or any project-specific files.

```
chore: upgrade framework to <to_version>

Updated from <from_version>. Skill commands, user-guide.md,
features/README.md, and FRAMEWORK_VERSION replaced wholesale.
See PR for CLAUDE.md / constitution.md structural changes
requiring manual review.
```

Push the branch: `git push -u origin <current-branch>`.

## Open PR

Open a pull request against `main` using `mcp__github__create_pull_request`:

- **Title:** `chore: upgrade framework to <to_version>`
- **Body:**

```
## Upgraded files

`.claude/commands/*.md`, `user-guide.md`, `features/README.md`, `FRAMEWORK_VERSION`
upgraded from `<from_version>` to `<to_version>`.

## Manual review checklist

<structural diff output as described above>

## Verification

Run the test suite to confirm no regressions. Feature artifacts and project-specific
configuration (CLAUDE.md, constitution.md project sections, declaration.md) are unchanged.
```

- Ready for review (not draft). No reviewers or assignees, per CLAUDE.md (the owner is the PR author).

Present the PR URL to the user.
