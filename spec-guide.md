# Spec Guide

*The contract for authoring a feature spec upstream — in a claude.ai chat session with MCP access to this repo — so `/ship` can take it from committed markdown to a deployed release without re-litigating intent. If you are the chat session authoring the spec, this file is your instruction set.*

The spec is a boundary input: it says **what** to build and **what "done" means**, at the level of intent. It does not prescribe how the feature is built — `/ship` and the model own the path. Keep it to what only you can supply.

## Before drafting

Read, via MCP:

- `declaration.md` — what the project is, why, what's out of scope, the Platform, the Shape (named components/seams), and the Roadmap (the backlog; picking the next item happens in this conversation).
- `constitution.md` — the standards that apply, the patterns in use, the **Spec-authoring lessons** (mistakes previous builds already taught — do not repeat them), the acknowledged risks this feature might compound, and the **Acceptance bar** `/ship` will hold the build to.
- Any precedent repos or in-repo reference material named in `CLAUDE.md` — that is ground truth; read it before assuming a pattern.

On a fresh project, `/setup` must have merged first — there's nothing sound to author against until the declaration exists.

## Picking the next feature (the backlog)

`declaration.md`'s Roadmap is the backlog, and this conversation is where it gets worked. Before proposing anything, sort each Roadmap entry by checking `features/`: **built** (folder exists, shipped — mention as progress), **spec'd but not shipped** (a locked contract waiting on `/ship` — a heads-up, never pickable), **unspec'd** (the only selectable candidates). Open with a short read on state — the project in a sentence, what's built, anything waiting on `/ship`, then the unspec'd items in Roadmap order with a one-line read on size and readiness — then ask which is next. Don't pick for the owner. If they pick something off-Roadmap, that's fine; ask whether the Roadmap should be updated so the recorded sequence stays honest. Roadmap edits happen only with explicit approval.

## Where the spec lands

`features/[feature-name]-[number]/spec.md`, committed to `main`. Numbers are sequential per feature name; never overwrite a previous version — create a new numbered folder. Design artifacts (below) are committed alongside and referenced from the spec.

## The shape of a spec

**Intent** — What / Why / Success / Out of scope, at the level of intent, not implementation.

**Behavioral requirements** — user stories with acceptance criteria (a user, a need, the criteria that confirm it works), plus edge cases and failure modes. Every requirement must be *testable as written*: `/ship` derives the acceptance suite directly from this section, so a requirement that can't be verified by an observable behavior needs rewriting until it can. This is the property that lets the spec serve as the definition of done.

**Design** — the components, the seams between them, and the behavioral properties each must hold.

- **Constraint discipline.** Every constraint must be behavioral — a property the user or system cares about ("requests time out within 5 s"), never a call signature, attribute name, or code shape. The single exception: when the call shape itself is the contract (a public interface this project exposes to callers), name it and say why.
- **Pattern reuse.** If a component reuses a pattern already in the constitution's registry, mark it explicitly — `Reuses pattern: <name>`. Only mark genuine, complete reuse.
- When a deploy step mutates an asynchronously-updating cloud resource, sequence the wait/poll between dependent mutations and name any value shared between bootstrap and deploy scripts as an explicit contract.

**Design artifacts (optional)** — for UI-shaped features, Claude Design output (mockups, design specs) committed alongside the spec and referenced from it. Backend, CLI, and service work skips this entirely; the framework never requires it.

## Sizing a feature

One spec is one coherent unit of value. Split into separate specs only when the parts carry **independent value or independent risk** — when shipping one without the other makes sense, or when they'd be reviewed against different concerns. Do not split a single intention just because its description contains an "and"; a spec shredded into artificial fragments generates more process than it saves. When in doubt, keep it together and let the build decompose internally.

## What not to include

- **Implementation prescriptions** — file layouts, library call shapes, internal signatures. `/ship` owns implementation.
- **Test code.** `/ship` writes the acceptance suite in its own context — a deliberate separation, so the spec's author and the tests' author differ.

## Handoff

Commit the spec (and any artifacts) to `main`. Then the owner opens a Claude Code cloud session on the repo and runs:

```
/ship feature-name: [name]
```

`/ship` asks its questions once, up front. The tighter the spec, the emptier that batch.
