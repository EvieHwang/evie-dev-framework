# Evie Dev Framework

A template repository for building software with Claude Code. Slash commands produce specification artifacts and then build against them, scaling from small bounded changes up to full feature builds.

## This repo is a template

Clone or fork it; the clone becomes the app you're building. Several files ship with placeholders that get filled in as you use the template:

- **This `README.md`** — describes the framework; replace it with one for your app.
- `CLAUDE.md` — `# [Project Name]` and the run/test/deploy sections.
- `declaration.md` — what / why / for whom / out of scope / Shape / Roadmap.
- `constitution.md`'s `## Testing` section — framework and run command, populated on first test generation.
- `features/` — empty; folders are created per feature as commands run.

The template itself has no executable code and no tests; the quality gates in `constitution.md` apply to copied apps, not to this template.

## Getting started

Open the cloned repo in Claude Code and:

1. `/setup` — fills `CLAUDE.md` (name, description, run/test, deploy target) and writes a README stub.
2. `/declaration` — populates `declaration.md` (intent + Shape + Roadmap). Runs automatically as the second phase of `/setup` on a fresh clone.
3. Pick your first piece of work and run its command.

## Commands at a glance

| Command | Use it for |
|---|---|
| `/setup` | One-time fill of `CLAUDE.md` + README stub on a fresh clone |
| `/declaration` | Write or refine the project declaration (intent, Shape, Roadmap) |
| `/patch` | A bounded change — bug, tweak, small feature — in one session |
| `/feature` | Declare a feature before speccing it |
| `/spec` | Drive requirements → architecture → adversarial → tests → DAG automatically |
| `/requirements`, `/architecture`, `/adversarial`, `/tests`, `/dag` | The spec steps, run manually for closer involvement |
| `/build` | Execute the DAG wave-by-wave and open a PR |
| `/next` | Run one build wave (resume a partial DAG) |
| `/retro` | Capture what was learned after a feature build |
| `/upgrade` | Pull the latest framework files into this project |

See [`user-guide.md`](user-guide.md) for how the requirements ↔ architecture loop converges, how adversarial findings flow back, and how the DAG-driven build works.

## Development environment

Designed for Claude Code in the cloud sandbox attached to this GitHub repo. The sandbox provisions a working branch per session; commands commit to it and open PRs against `main` when work is complete. Local Claude Code works too — the only sandbox-specific accommodation is that user-global preferences live inside `CLAUDE.md` rather than `~/.claude/CLAUDE.md` (which doesn't exist in the sandbox).
