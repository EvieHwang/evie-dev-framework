# Evie Dev Framework

A template repository for building software with Claude Code. It encodes the part of building that the model can't derive — what to build, why, what "done" means, the standards, the risks knowingly accepted — and hands the rest (planning, decomposition, execution) to the runtime.

## This repo is a template

Clone or fork it; the clone becomes the app you're building. Several files ship with placeholders that get filled in as you use the template:

- **This `README.md`** — describes the framework; replace it with one for your app.
- `CLAUDE.md` — `# [Project Name]` and the run/test/deploy sections.
- `declaration.md` — what / why / for whom / out of scope / Shape / Roadmap.
- `constitution.md`'s `## Testing` section — framework and run command, populated on first spec.
- `features/` — empty; folders are created per feature as commands run.

The template itself has no executable code and no tests; the quality gates in `constitution.md` apply to copied apps, not to this template.

## The two jobs, treated oppositely

This framework does two separable things:

1. **Encodes human judgment** — intent, scope, the definition of done, standards, accepted risks. The model can't derive any of it. The framework keeps and protects this.
2. **Manages execution** — planning, decomposition, iteration. The runtime (Opus 4.8 + Dynamic Workflows) now does this natively, so the framework hands it over instead of hand-walking it.

The whole command set follows from that split. Commands that carry the owner's intent survive; machinery that existed only to supervise the model's process is gone. **Tests are the deliberate exception that lives on both sides** — they are the executable half of "what done means," so the framework makes them more central, not less.

## Getting started

Open the cloned repo in Claude Code and:

1. `/setup` — fills `CLAUDE.md` (name, description, run/test, deploy target) and writes a README stub, then runs `/declaration`.
2. Pick your first piece of work and run its command.

## Commands at a glance

| Command | Use it for |
|---|---|
| `/setup` | One-time fill of `CLAUDE.md` + declaration on a fresh clone |
| `/declaration` | Write or refine the project declaration (intent, Shape, Roadmap) |
| `/feature` | Coached scoping pass when a feature's shape is genuinely fuzzy (optional) |
| `/spec` | Produce a feature's spec + test suite, then run an independent adversarial gate |
| `/build` | Implement against the spec's tests until the suite passes, then open a PR |
| `/patch` | A bounded change — bug, tweak, small feature — in one session |
| `/retro` | Capture what was learned after a feature build |
| `/upgrade` | Pull the latest framework files into this project |

The main path is `/spec` → merge → `/build`. Everything else is upstream intent (`/declaration`, `/feature`) or utility. See [`user-guide.md`](user-guide.md) for the full division of labor.

## What changed in V2

V2 collapsed a hand-built orchestration layer that the runtime now provides. See [`MIGRATION.md`](MIGRATION.md) for the thesis and the full before/after.

## Development environment

Designed for Claude Code in the cloud sandbox attached to this GitHub repo. The sandbox provisions a working branch per session; commands commit to it and open PRs against `main` when work is complete. Local Claude Code works too — the only sandbox-specific accommodation is that user-global preferences live inside `CLAUDE.md` rather than `~/.claude/CLAUDE.md` (which doesn't exist in the sandbox).
