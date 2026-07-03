# Spec Guide

*The contract for authoring a feature spec upstream — in a claude.ai chat session with MCP access to this repo — so `/ship` can take it from committed markdown to deployed release without re-litigating intent. If you are the chat session authoring the spec, this file is your instruction set.*

## Before drafting

Read, via MCP:

- `declaration.md` — what the project is, why, what's out of scope, the Platform, the Shape (named components/seams), and the Roadmap. The Roadmap is the backlog: picking and pressure-testing the next item happens in this conversation.
- `constitution.md` — the standards that apply by default, the patterns in use, the **Spec-authoring lessons** (mistakes previous builds already taught — do not repeat them), the acknowledged risks this feature might compound, and the Build contract `/ship` will hold the spec to.
- Any precedent repos or in-repo reference material named in `CLAUDE.md` — that is ground truth; read it before assuming a pattern.

On a fresh project, `/setup` must have merged first — a walking-skeleton spec is derived from the declaration's Shape, so there is nothing sound to author against until the declaration exists.

## Where the spec lands

`features/[feature-name]-[number]/spec.md`, committed to `main`. Numbers are sequential per feature name; never overwrite a previous version — create a new numbered folder. Design artifacts (see below) are committed alongside in the same folder and referenced from the spec.

## The shape of a spec

**Intent** — What / Why / Success / Out of scope, at the level of intent, not implementation. If this is the project's **first** feature, it should be a walking skeleton: the thinnest vertical slice that exercises the Shape's seams end to end, with most behavior stubbed — not the most valuable feature. Depth comes from feature 2 onward.

**Behavioral requirements** — user stories with acceptance criteria (a user, a need, the criteria that confirm it works), plus edge cases and failure modes. Every requirement must be *testable as written*: `/ship` derives the acceptance suite directly from this section, so a requirement that can't be verified by an observable behavior needs rewriting until it can.

**Design** — the components, the seams between them, and the behavioral properties each must hold.

- **Constraint discipline.** Every constraint must be behavioral — a property the user or system cares about ("requests time out within 5 s"), never a call signature, attribute name, or code shape. The single exception: when the call shape itself is the contract (a public interface this project exposes to callers), name it and say why.
- **Pattern reuse.** If a component reuses a pattern already in the constitution's registry (existing auth module, established deploy workflow), mark it explicitly — `Reuses pattern: <name>`. Only mark genuine, complete reuse.
- When a deploy step mutates an asynchronously-updating cloud resource, sequence the wait/poll between dependent mutations and name any value shared between bootstrap and deploy scripts as an explicit contract.

**Design artifacts (optional)** — for UI-shaped features, Claude Design output (mockups, design specs) committed alongside the spec and referenced from it. Backend, CLI, and service work skips this entirely; the framework never requires it. On a first feature, scope the design run per `## Design on the first feature` below.

## Design on the first feature (walking skeleton)

A skeleton gets **structural** design only — the frame, not the rooms:

- **In scope:** the app shell (layout structure, navigation pattern), the visual system (type scale, spacing, color, dark mode) that later screens inherit, and one deliberately plain placeholder screen per Shape area.
- **Out of scope:** screen-level content, states, flows, and polish. The stubs are replaced feature by feature from feature 2 onward — designing them now is designing rooms about to be remodeled. From feature 2 on, design-per-feature is the normal rhythm for anything UI-shaped.
- If the skeleton has no meaningful UI surface, skip design entirely.

Run Claude Design with this prompt, filling the brackets from `declaration.md` and the constitution's UI aesthetic:

> I'm designing the shell of a new app, not its screens. This is for a walking skeleton — the first build threads every part of the app end to end with stubbed behavior; individual screens get their real design later, one feature at a time.
>
> Design only: (1) the **app shell** — layout structure, navigation pattern, and how a user moves between the main areas; (2) the **visual system** — type scale, spacing, color and dark mode — so later screens inherit a system instead of inventing their own; (3) **one deliberately plain placeholder screen per area**: real title, real navigation, stub body that makes clear what will eventually live there. Do not design the content, states, or flows of these screens.
>
> The app's main areas are: [Shape components]. Platform: [Platform]. Constraints: [the constitution's UI aesthetic — e.g. information-dense, 14px base, tight line heights, dark mode as a first-class surface].
>
> If you find yourself refining a screen's content or adding states beyond empty-stub, stop — that's a later feature's job.

Commit the output to `features/[feature-name]-[number]/design/` and add this line to the spec's Design section: *"Design artifacts define the shell and visual system only; screen-level design is explicitly out of scope for this feature."* That sentence tells `/ship`'s adversarial gate not to flag unstyled stubs as coverage gaps, and tells the build not to invent polish the spec didn't ask for.

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
