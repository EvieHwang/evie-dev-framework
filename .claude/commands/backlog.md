---
description: Kicks off a conversation to pick the next thing to build. Reads declaration.md's Roadmap, cross-references what's already been built under features/, surfaces the open backlog items, and stops for you to choose one — then discusses it until you're ready to run /spec. Read-only: writes nothing and changes no other part of the workflow.
---

This command only starts a conversation. It produces no artifacts, edits no files, and is not a step in the mainline — it is a way to look at the backlog and pick the next item before `/spec`.

Read these before saying anything:
- `declaration.md` — especially the **Roadmap** section, which is the backlog.
- `constitution.md` — standards, patterns, principles, acknowledged risks that might constrain a candidate.
- `features/` — skim the existing feature folders so you can tell which Roadmap entries have already been built or are in flight, and don't present finished work as open.

Then give the user a short read on the current state:
- What the project is, in a sentence (from `declaration.md`).
- What's already been built or is in flight (from `features/`).
- The **candidate backlog items still open** — a brief numbered list, each with a one-line description plus your read on its size, dependencies, and readiness. Order them by the Roadmap sequence unless something obviously blocks an earlier item.

**Then stop and ask which item is next.** Do not pick for the user. Raise it through the `AskUserQuestion` tool with the candidate items as options (keep "Other" open for something not on the list) — in a cloud/web/mobile session that question is what notifies the owner the run is parked on their input. This is the command's only stop.

If `declaration.md` has no Roadmap section (older project, or a fresh clone that hasn't run `/setup`), say so plainly: there is no backlog to review yet. Recommend `/declaration` to add a Roadmap, or `/setup` on a fresh clone, and stop.

Once the user names the item, discuss it — conversationally, at the level of intent:
- Scope: what's in, what's out.
- Open questions that need answering before a spec could be written.
- What "done" looks like for this item.
- Anything in `constitution.md` or the existing `features/` that constrains or informs it.

Keep going until the user is satisfied the item is well understood. Do **not** run `/spec`, `/feature`, or any other command — kicking off the next step is the user's call. When the discussion has converged, say so and note that the next move is theirs (typically `/spec`, or `/feature` first if the scope still feels fuzzy).

Exit condition: the user has picked the next backlog item and talked it through to their satisfaction. Nothing is written, committed, or pushed.
