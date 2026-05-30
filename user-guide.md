# User Guide

A build framework for Claude Code, distributed as a template repository. It provides slash commands (in `.claude/commands/`) that produce specification artifacts in `features/[name]-[number]/` and then build against them. The artifacts are the contract between what you wanted and what got built.

This guide is the human's map: which command to reach for, and how the pieces fit. Each command's exact procedure lives in its own file in `.claude/commands/` — that file is the source of truth, and you can read it if you want the mechanics.

---

## Choosing your approach

**Quick change — `/patch`.** Bug fixes, tweaks, polish, small features where the intent is clear and the work fits one session. No feature artifacts; the PR body is the documentation. Use when you know what correct looks like and the change is contained.

**Feature build — `/spec` → `/build`.** Real features built from a written spec. `/spec` produces the feature declaration inline from the chat that led up to it, then runs the full pre-build pipeline (requirements → architecture → adversarial → tests → DAG → PM summary → PR). A DAG-driven build then executes wave-by-wave. Use when the work is deliberate, spans multiple components, or carries hard-to-reverse architectural decisions.

---

## Core documents

- **`declaration.md`** — the project's statement of intent: what it is, why, for whom, what it explicitly doesn't do, plus **Shape** (3–7 named components/seams) and **Roadmap** (3–7 anticipated features in rough order). Shape and Roadmap are revisable as the project learns.
- **`constitution.md`** — the project's accumulated judgment: standards registry, architectural principles, patterns in use, quality gates, testing framework, decision log, acknowledged risks.
- **`CLAUDE.md`** — the app's operational map: repo layout, run/test/deploy context, and user-global identity coordinates.

Every command reads `constitution.md` and `declaration.md` before any architectural decision, and the relevant feature artifacts before acting. This is wired into each command — you don't have to manage it.

---

## Running a quick change

```
/patch
```

Identifies the minimal change, presents a plan with breakage risk surfaced, waits for approval, implements only that, runs existing tests, and opens a PR. The PR body (Intent / Change / Verification / Risk) is the record.

---

## Running a feature build

The pipeline has a pre-build sequence, two iterative loops, and a DAG-driven build. Artifacts get committed as they're produced.

### Pre-build: orchestrated or manual

Both paths produce identical artifacts — the difference is how involved you are between steps. `/spec` detects which artifacts already exist and resumes from the right point, so you can switch between the two mid-pipeline.

**Orchestrated (recommended for most features)**

```
/declaration                        # one-time per project
... chat about the next feature ...
/spec      feature-name: [name]
```

`/spec` starts by producing `declaration.md` inline from the preceding conversation (Stage 0 — coached only in walking-skeleton mode), then drives requirements → architecture → adversarial → tests → DAG automatically, using focused sub-agents so each artifact gets full-context quality. It stops only when a decision needs product judgment: scope drift, declaration tension, an unresolvable HIGH finding, or a risk acknowledgment only you can make. It ends with `spec-summary.md`, a PM-level report. Read it, start a new session, run `/build`.

**Optional — `/feature` for fuzzy scope**

```
/feature   feature-name: [name]
/spec      feature-name: [name]
```

Reach for `/feature` when the scope is genuinely unclear and you want a coached scoping pass before the spec pipeline runs. `/spec` will detect the populated `declaration.md` and skip Stage 0.

**Manual (for refinement or closer involvement)**

```
/declaration                        # one-time per project
/feature       feature-name: [name]   # optional — produces the feature declaration
/requirements  feature-name: [name]
/architecture  feature-name: [name]
... iterate requirements ↔ architecture until convergence
/adversarial   feature-name: [name]
... address findings via the loop; re-run adversarial
/tests         feature-name: [name]
/dag           feature-name: [name]
```

Run manually when you want to review and shape each artifact before the next step. If `/feature` is skipped, the feature `declaration.md` must be authored before `/requirements` — the requirements skill reads it as its anchor.

### How the loops work

**Requirements ↔ architecture.** Each side writes a stability marker when it has nothing new to surface. The loop has converged when the latest `requirements.md` and `design.md` both carry their markers. Two passes is common; once is enough for trivial work; more when committed architecture surfaces new constraints.

**Adversarial ↔ req/arch.** `/adversarial` reviews requirements + design and produces `adversarial-review.md`, a living artifact where every finding has a stable ID, severity, and status. Findings route back to `/requirements` or `/architecture`, which address them and mark them `addressed`; the next `/adversarial` run verifies and promotes to `resolved`. Zero findings is a valid outcome. Acknowledged findings propagate to the constitution's risk table.

### Build

```
/build   feature-name: [name]
```

Drives the DAG wave-by-wave (via `/next` internally) until every task is complete, runs the full test suite, and opens a PR. `/next` is also user-invokable to resume a partial DAG after a session ended.

During build, tests are the source of truth and requirements are immutable; design is a recommendation. When implementation reality diverges from design (or a test contains a genuine error), the build agent records the deviation in `build-deviations.md` rather than silently editing the spec — those entries flow back into the adversarial loop on the next pass.

### Retro

```
/retro   feature-name: [name]
```

Optional but recommended. A coached conversation producing `retro.md` — what went well, what didn't, and concrete proposed changes to `constitution.md`, `CLAUDE.md`, and skill prompts, with project-vs-template lessons marked separately so template-level improvements lift directly into a framework-update PR.

---

## New project setup

After cloning the template:

1. `/setup` — coached fill of `CLAUDE.md` (name, description, run/test/deps, deployment target) and a README stub. Commits to the current branch.
2. `/declaration` — populate the project declaration (intent + Shape + Roadmap).
3. Edit `constitution.md`'s app-specific sections as the project develops; the template ships with universal defaults already filled in (prune anything that doesn't apply).
4. Pick an approach for your first piece of work.

`/setup` runs `/declaration` as its second phase, so steps 1–2 happen in one session on a fresh clone.

---

## Keeping the framework up to date

```
/upgrade
```

Fetches the latest framework-owned files and writes them into the current project, then opens a PR.

- **Replaced wholesale** (projects have no reason to customize these): `FRAMEWORK_VERSION`, `user-guide.md`, `features/README.md`, `.claude/commands/*.md`.
- **Never replaced wholesale** (mix framework template sections with project-specific content): `CLAUDE.md` and `constitution.md`. `/upgrade` compares the framework-owned sections and surfaces differences as a manual review checklist in the PR body — apply what's relevant, skip what isn't.

`FRAMEWORK_VERSION` holds the date the framework was last fetched; `/upgrade` reads it to detect whether an upgrade is needed. To force a same-day re-upgrade, delete the file locally first.

**Bootstrapping a pre-`/upgrade` project:** open a session on the old project and ask Claude to `curl` the upgrade command into place:

```
curl -sf https://raw.githubusercontent.com/EvieHwang/evie-dev-framework/main/.claude/commands/upgrade.md -o .claude/commands/upgrade.md
```

Then run `/upgrade` to pull in the rest.
