# Spec Guide

*The contract for authoring a feature spec upstream — in a claude.ai chat session with MCP access to this repo — so `/ship` can take it from committed markdown to deployed release without re-litigating intent. If you are the chat session authoring the spec, this file is your instruction set.*

## Before drafting

Read, via MCP:

- `declaration.md` — what the project is, why, what's out of scope, the Platform, the Shape (named components/seams), and the Roadmap. The Roadmap is the backlog: picking and pressure-testing the next item happens in this conversation.
- `constitution.md` — the standards that apply by default, the patterns in use, the **Spec-authoring lessons** (mistakes previous builds already taught — do not repeat them), the acknowledged risks this feature might compound, and the Build contract `/ship` will hold the spec to.
- Any precedent repos or in-repo reference material named in `CLAUDE.md` — that is ground truth; read it before assuming a pattern.

## Where the spec lands

`features/[feature-name]-[number]/spec.md`, committed to `main`. Numbers are sequential per feature name; never overwrite a previous version — create a new numbered folder. Design artifacts (see below) are committed alongside in the same folder and referenced from the spec.

## The shape of a spec

**Intent** — What / Why / Success / Out of scope, at the level of intent, not implementation. If this is the project's **first** feature, it should be a walking skeleton: the thinnest vertical slice that exercises the Shape's seams end to end, with most behavior stubbed — not the most valuable feature. Depth comes from feature 2 onward.

**Behavioral requirements** — user stories with acceptance criteria (a user, a need, the criteria that confirm it works), plus edge cases and failure modes. Every requirement must be *testable as written*: `/ship` derives the acceptance suite directly from this section, so a requirement that can't be verified by an observable behavior needs rewriting until it can.

**Design** — the components, the seams between them, and the behavioral properties each must hold.

- **Constraint discipline.** Every constraint must be behavioral — a property the user or system cares about ("requests time out within 5 s"), never a call signature, attribute name, or code shape. The single exception: when the call shape itself is the contract (a public interface this project exposes to callers), name it and say why.
- **Pattern reuse.** If a component reuses a pattern already in the constitution's registry (existing auth module, established deploy workflow), mark it explicitly — `Reuses pattern: <name>`. Only mark genuine, complete reuse.
- When a deploy step mutates an asynchronously-updating cloud resource, sequence the wait/poll between dependent mutations and name any value shared between bootstrap and deploy scripts as an explicit contract.

**Design artifacts (optional)** — for UI-shaped features, Claude Design output (mockups, design specs) committed alongside the spec and referenced from it. Backend, CLI, and service work skips this entirely; the framework never requires it.

## What not to include

- **Implementation prescriptions** — file layouts, library call shapes, internal signatures. `/ship` owns implementation; the gate flags a spec that locks it in.
- **Test code.** `/ship` writes the acceptance suite in its own session — a deliberate separation, so the spec's author and the tests' author are different contexts.
- **Multiple features.** If the value can't be described in one sentence without "and" doing heavy lifting, split it into separate specs.

## Handoff

Commit the spec (and any artifacts) to `main`. Then the owner opens a Claude Code cloud session on the repo and runs:

```
/ship feature-name: [name]
```

`/ship` asks its questions once, up front. The tighter the spec, the emptier that batch.
