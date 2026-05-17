# User Guide

A three-tier build framework for Claude Code, distributed as a template repository. Each tier matches a different size of work: T1 for bounded changes that fit a single session, T2 for features that need a written spec before building, T3 for work too large to hold in one context window. Choose the tier that matches the scope. The framework provides skill commands (in `.claude/commands/`) that produce specification artifacts in `features/[name]-[number]/` and then build against them. The artifacts are the contract between what you wanted and what got built.

---

## The three tiers

**Tier 1 — small bounded change.** Bug fixes, tweaks, polish, small features where the intent is clear and the work fits in a single session. No feature artifacts are produced. The PR body is the documentation. Use when you know what correct looks like and the change is contained.

**Tier 2 — feature build.** Real features built from a written spec. A four-skill sequence produces intent → plan → verify (tests) → build. All four skills run in a single context window. Use when the work is deliberate but contained — you could complete it in a focused session if the spec were already written.

**Tier 3 — full system build.** Large features or full applications that don't fit in one context window. A pre-build sequence (declaration → requirements → design → adversarial review → DAG) produces committed artifacts. A DAG-driven build then executes across multiple sessions, advancing wave-by-wave. Use when the work spans multiple components, requires hard-to-reverse architectural decisions, or won't complete in one focused session.

---

## Repo structure

```
/
├── CLAUDE.md              # operational map for the app
├── declaration.md         # what this project is and why
├── constitution.md        # principles, standards, decisions, patterns
├── README.md              # human-facing project readme
│
├── features/
│   └── [name]-[number]/   # one folder per feature
│       ├── (T2: intent.md, plan.md, verify.md, tests/)
│       └── (T3: declaration.md, requirements.md, design.md,
│                adversarial-review.md, dag.md, state.md,
│                verify.md, tests/)
│
└── .claude/
    └── commands/          # all skill commands
        ├── declaration.md
        ├── t1-build.md
        ├── t2-intent.md, t2-plan.md, t2-verify.md, t2-build.md
        ├── t3-feature-declaration.md, t3-requirements.md,
        │   t3-architecture.md, t3-adversarial.md, t3-generate-dag.md,
        │   t3-test-coach.md, t3-next-step.md, t3-build.md
```

Feature folders are named `[feature-name]-[number]`. Numbers are sequential per feature name and disambiguate iterations — never overwrite a previous version, create a new numbered folder. Artifacts inside feature folders are populated by skills at runtime; never pre-create empty placeholders (several skills detect their pass number by checking which artifacts exist).

---

## Core documents

**`declaration.md`** — the project's statement of intent. What the project is, why it exists, for whom, and what it explicitly does not do. Written once at project setup. Held stable; revising it mid-build usually means it was written at the wrong level of abstraction.

**`constitution.md`** — the project's accumulated judgment. Contains: the standards registry (Apple HIG, WCAG, OWASP, OpenAPI), architectural principles, patterns in use (frontend stack, commit style, service layout), quality gates that must hold before any PR, the testing framework and run command (populated on first test generation), the state-file format spec, and the decision log.

**`CLAUDE.md`** — the app's operational map. Repo layout pointers, read-before-build discipline lines, run/test/deploy context for whichever target the app uses, and user-global identity coordinates (carried in-file because the cloud sandbox has no `~/.claude/CLAUDE.md`).

Every skill reads constitution.md and declaration.md before architectural decisions. Most skills read the relevant feature artifacts in order.

---

## Standards registry

Lives in `constitution.md` under `## Standards`. Default entries:

| Domain | Standard |
|---|---|
| Interface | Apple Human Interface Guidelines |
| Accessibility | WCAG 2.1 AA |
| Security | OWASP Top 10 |
| API contracts | OpenAPI Specification |

Extended entries available when relevant: Microsoft REST API Guidelines (external APIs), OWASP Top 10 for LLMs (AI integration), OWASP ASVS (sensitive data).

Skills follow these standards without deviation unless `declaration.md` explicitly requires otherwise. Any deviation must be surfaced as a decision, not made silently.

---

## Running Tier 1

```
/t1-build
```

The skill reads CLAUDE.md and constitution.md, inspects the affected module (and any existing feature artifacts if this is refining T2/T3 output), proposes a minimal plan with breakage risk surfaced, waits for approval, implements only the minimal change, runs existing tests, and opens a PR.

Test failures get one remediation attempt; a second failure stops and reports.

The PR body has four sections (Intent / Change / Verification / Risk) — this is the record of the change.

---

## Running Tier 2

Four skills in sequence. Each commits its artifact to the current branch.

```
/t2-intent     feature-name: [name]
/t2-plan       feature-name: [name]
/t2-verify     feature-name: [name]
/t2-build      feature-name: [name]
```

**`t2-intent`** — coached conversation, then writes `intent.md` (Purpose, User stories, Out of scope). Confirm before output; review the written doc before continuing.

**`t2-plan`** — reads intent. Surfaces architectural considerations and the implementation approach. If a consideration reshapes the intent, the skill stops and the user updates intent.md before re-running. This feedback loop is the primary quality lever in T2.

**`t2-verify`** — derives behavioral tests (from intent) and structural tests (from plan). On first use in a project, the skill picks a test framework and writes it into constitution.md's `## Testing` section. Outputs a human-readable `verify.md` summary plus executable test files. Untestable requirements stop the skill — fix intent or plan first.

**`t2-build`** — implements until every test passes. Treats tests as immutable spec (never edits them to make them pass). On a test failure, attempts one remediation, then stops. Opens a PR with sections for Intent / Plan / Verification / Risk.

---

## Running Tier 3

T3 has a pre-build sequence, two iterative loops, and a DAG-driven build. The pre-build artifacts get committed as they're produced.

### Pre-build sequence

```
/declaration                        # one-time per project
/t3-feature-declaration  feature-name: [name]
/t3-requirements         feature-name: [name]
/t3-architecture         feature-name: [name]
... (iterate requirements ↔ architecture until convergence)
/t3-adversarial          feature-name: [name]
... (address findings via the loop; re-run adversarial)
/t3-generate-dag         feature-name: [name]
/t3-test-coach           feature-name: [name]
```

**`declaration`** — project-level coach. Produces `declaration.md` at repo root. One-time at setup; re-runnable to refine. Skip if declaration.md is already current.

**`t3-feature-declaration`** — coaches a feature-level declaration anchored to the project declaration. Produces `features/[name]-[number]/declaration.md` (What / Why / Success / Out of scope).

**The requirements ↔ architecture loop.** Each skill writes a stability marker at the bottom of its output when it has no new flags to surface (`Requirements stable — no architectural feedback to incorporate`; `Architecture stable — no requirements changes flagged`). The loop has converged when both the most recent `requirements.md` and `design.md` carry their markers. Iterate as many times as needed — twice is common, once is enough for trivial work, more is needed when committed architecture surfaces new constraints.

**`t3-adversarial`** — reviews requirements + design through six lenses (integrity, coverage, security, standards compliance, failure modes, scope drift). Outputs `adversarial-review.md` as a living artifact: every finding has a stable ID (F-001, F-002…), severity (HIGH / MEDIUM / LOW), a recommended action naming which skill should address it, and a status (`open` / `addressed` / `acknowledged` / `resolved` / `deferred`).

When findings exist, the user re-enters the req↔arch loop. `t3-requirements` and `t3-architecture` each read `adversarial-review.md`, address every `open` finding routed to them, and mark those findings `addressed`. The next run of `t3-adversarial` verifies the claims and promotes verified findings to `resolved`.

**`t3-generate-dag`** — gates on (a) both req and arch reporting stable and (b) no `open` HIGH findings in adversarial review. Produces `dag.md` (tasks with ID, description, inputs, outputs, dependencies, wave, acceptance condition) grouped into parallel waves, and initializes `state.md` with every task `pending`.

**`t3-test-coach`** — generates tests in the chosen framework, tagged by DAG task ID so the build agent knows which tests cover which task. Every task in `dag.md` must have at least one test. If tests already exist from a prior generation, they get regenerated (the prior tests were tied to the prior spec).

### Build

```
/t3-build      feature-name: [name]
```

Orchestrates the DAG. Internally invokes `/t3-next-step` in a loop until `state.md` shows every task complete, then runs the full test suite and opens a PR. The PR body has sections for Feature declaration / Requirements / Design / Adversarial review / Build summary / Risk.

`/t3-next-step` is the per-wave executor — it's user-invokable to resume a partial DAG after a session ended, or for manual single-wave execution. One wave per invocation matches the cloud-sandbox session boundary, so `state.md` is always durable at a clean checkpoint.

Tasks within a wave may run in parallel via sub-agents. Tests tagged to those task IDs run after the wave; a failed test sets the responsible task back to `failed` and stops the loop.

Build skills never modify requirements, design, dag, or tests. If something looks wrong during build, the skill stops and surfaces it — the resolution is to update upstream artifacts and regenerate the DAG, not edit in place.

---

## New project setup

After cloning the template:

1. Fill in `CLAUDE.md` — project name, one-line description, run/test/deps commands, deployment target (uncomment one of Eviebot / AWS / Apple-Xcode and delete the rest).
2. Run `/declaration` to populate the project declaration.
3. Edit `constitution.md`'s app-specific sections (architectural principles, patterns, quality gates, out-of-scope) as the project develops; the template ships with universal defaults already populated.
4. Pick a tier for your first piece of work and run the corresponding command.

The standards registry, default principles, and patterns ship pre-filled. You can prune anything that doesn't apply to this app (e.g., remove the frontend stack entry for a backend-only project).

---

## Command reference

| Command | Reads | Produces |
|---|---|---|
| `/t1-build` | constitution, CLAUDE | PR (no feature artifacts) |
| `/t2-intent` | constitution, declaration | `features/[name]-[#]/intent.md` |
| `/t2-plan` | constitution, declaration, intent | `features/[name]-[#]/plan.md` |
| `/t2-verify` | constitution, declaration, intent, plan | `verify.md`, `tests/`, populates `constitution.md` `## Testing` on first use |
| `/t2-build` | all of the above | code, commits, PR |
| `/declaration` | declaration | `declaration.md` (project root) |
| `/t3-feature-declaration` | declaration, constitution | `features/[name]-[#]/declaration.md` |
| `/t3-requirements` | constitution, declarations, design (if exists), adversarial-review (if exists) | `requirements.md` with stability marker |
| `/t3-architecture` | constitution, declarations, requirements, design (if exists), adversarial-review (if exists) | `design.md` with stability marker |
| `/t3-adversarial` | constitution, declarations, requirements, design, prior adversarial-review (if exists) | `adversarial-review.md` (living artifact) |
| `/t3-generate-dag` | all pre-build artifacts | `dag.md`, `state.md` |
| `/t3-test-coach` | constitution, declarations, requirements, design, dag | `verify.md`, `tests/` |
| `/t3-build` | all pre-build + state + verify | drives DAG; final tests; PR |
| `/t3-next-step` | constitution, dag, state, verify | executes one wave; updates state |
