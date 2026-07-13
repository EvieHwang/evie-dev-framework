# Evie Dev Framework

A template repository for building software with Claude Code. It encodes the part of building that the model can't derive — what to build, why, what "done" means, the standards, the risks knowingly accepted — and then gets out of the way. It specifies the edges and the definition of done, not the path between them: how the feature actually gets built is left to the model, which keeps getting better at exactly that.

## The workflow

1. **Author the spec upstream.** In a claude.ai chat with MCP access to the repo, talk through the next feature. The chat reads `declaration.md` and `constitution.md`, follows `spec-guide.md`, and commits `features/<name>-<number>/spec.md` to `main` — with Claude Design artifacts alongside when the feature is UI-shaped.
2. **Run `/ship` in a Claude Code cloud session.** It checks the spec against the project's accumulated judgment, asks everything unclear in one upfront batch, builds the feature to the constitution's acceptance bar (behavior real, tests green, gates met), gets an independent risk review sized to the feature's stakes, and opens the PR.
3. **Approve the PR.** Your review is the one human gate mid-flow; merging acknowledges any carried findings listed in the PR body.
4. **The session watches the release.** After merge it follows the deploy through GitHub Actions and the target platform to a verified healthy release, fixes what breaks, then reports and ends.

## This repo is a template

Clone or fork it; the clone becomes the app you're building. Files that ship with placeholders: this `README.md` (replace with your app's), `CLAUDE.md` (project name, run/test/deploy sections), `declaration.md`, `constitution.md`'s `## Testing` section (populated on first `/ship`), and `features/` (empty; folders are created per feature). The template itself has no executable code; the quality gates in `constitution.md` apply to copied apps.

## Commands

| Command | Use it for |
|---|---|
| `/setup` | One-time fill of `CLAUDE.md` + `declaration.md` on a fresh clone |
| `/ship` | Everything between a committed spec and a verified deployment |
| `/upgrade` | Pull the latest framework files into this project |

Small changes that don't warrant a spec don't need a command — open a session and ask; the PR is the documentation.

## Where the judgment lives

- `declaration.md` — intent: what, why, for whom, out of scope, Platform, Shape, Roadmap.
- `constitution.md` — accumulated judgment: standards, principles, patterns, the acceptance bar (definition of done), spec-authoring lessons, quality gates, testing wiring, decision log, acknowledged risks.
- `spec-guide.md` — the contract the upstream chat follows when authoring a spec.
- `chat-skill/` — the canonical source of the claude.ai `spec-author` skill, a thin wrapper that fetches and follows `spec-guide.md` via MCP (uploaded once to claude.ai; not distributed by `/upgrade`).

See [`user-guide.md`](user-guide.md) for the full division of labor.
