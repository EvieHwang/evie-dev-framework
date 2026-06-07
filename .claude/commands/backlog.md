---
description: Kicks off a conversation to pick the next thing to build. Reads declaration.md's Roadmap, cross-references what's already been built under features/, surfaces the unspec'd backlog items, and stops for you to choose one — then discusses it until you're ready to run /spec. Read-only: writes nothing and changes no other part of the workflow.
---

This command only starts a conversation. It produces no artifacts, edits no files, and is not a step in the mainline — it is a way to look at the backlog and pick the next item before `/spec`.

Read these before saying anything:
- `declaration.md` — especially the **Roadmap** section, which is the backlog.
- `constitution.md` — standards, patterns, principles, acknowledged risks that might constrain a candidate.
- `features/` — skim the feature folders and sort each Roadmap entry into one of three states: **built** (don't surface it), **spec'd but not built** (a locked contract; its next step is `/build` in a fresh session — mention it as a heads-up, never as a pickable option), or **unspec'd** (no feature folder yet — these are the only selectable options).

Then give the user a short read on the current state: what the project is in a sentence, what's already built, any spec'd-but-not-built feature as a one-line reminder (it's waiting on its own `/build` session, not something to pick here), and then the **selectable options** — the unspec'd backlog items as a brief numbered list, each with a one-line description plus your read on size, dependencies, and readiness, ordered by the Roadmap sequence unless something obviously blocks an earlier item.

**Then stop and ask which unspec'd item is next** — via the `AskUserQuestion` tool, with the unspec'd items as options and "Other" kept open. Don't pick for the user. In a cloud/web/mobile session that question is what notifies the owner the run is parked on their input. This is the command's only stop.

If nothing unspec'd remains, or `declaration.md` has no Roadmap section at all, say so plainly and stop — there's nothing to take into `/spec` yet. Recommend `/declaration` to add or extend the Roadmap (or `/setup` on a fresh clone), and note any spec'd-but-not-built feature still waiting on its `/build` session.

Once the user names the item, discuss it conversationally at the level of intent: scope (what's in, what's out), open questions a spec would need to answer, what "done" looks like, and anything in `constitution.md` or `features/` that constrains it. Keep going until the user is satisfied the item is well understood.

Do **not** run `/spec`, `/feature`, or any other command — kicking off the next step is the user's call. When the discussion has converged, say so and note that the next move is theirs (typically `/spec`, or `/feature` first if scope still feels fuzzy).

Exit condition: the user has picked the next backlog item and talked it through to their satisfaction. Nothing is written, committed, or pushed.
