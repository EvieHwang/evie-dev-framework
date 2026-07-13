# Migration: V3 → V4 (2026-07-13)

*(Earlier migration records live in git history.)*

## The thesis

V3 collapsed the command pipeline into documents plus `/ship`, but `/ship` still spelled out **how to build** — a test-first ordering, `@frozen`/`@scaffolding` tagging, a walking-skeleton mandate, coverage maps, per-feature deviation files, and a rigid three-context separation. That middle is exactly the part the model is being actively made better at, so prescribing it is a depreciating bet: every prescribed step is an implicit claim the builder won't do it well unless told, and each model release makes more of those claims false.

V4 draws one line. The framework specifies the **edges** (constitution, spec, deployment contract) and the **definition of done** (a verifiable acceptance bar), and prescribes nothing about the path between them. The distinction it enforces everywhere:

- **Acceptance** — a verifiable output you're entitled to demand of a black box (tests green, gates pass, release verified healthy). Kept, and made the center of gravity.
- **Procedure** — dictating the internals (write tests first, tag them, build a skeleton, log deviations in this format). Dropped.

A useful side effect: process now right-sizes itself. Because `/ship` is asked for outputs proportional to the feature rather than a fixed sequence of steps, a trivial change produces little and a risky one produces more — without an explicit tier system.

## Kept

- **`constitution.md`** in full as the accumulated judgment and the home of the definition of done.
- **The independent risk review** — the one check a builder can't perform on itself. Now expressed as an acceptance guarantee ("surface an independent review of what was built") and **sized to the feature's stakes**, not run as fixed ceremony on everything. The security-fix re-check is kept.
- **Deployment-to-verified-health**, the acknowledged-risk propagation, and the spec-authoring-lessons feedback loop.
- **`/setup`**, **`/ship`**, **`/upgrade`** as the three commands.

## Changed

- **`## Build contract` → `## Acceptance bar (definition of done)`** in the constitution. Reframed from "how `/ship` treats the spec during the build" to "what `/ship` must be able to demonstrate for a feature to be done" — verifiable outputs only.
- **`/ship`** rewritten from six prescribed stages to intake → questions-once → build-to-the-bar → independent risk review → PR → watch-to-healthy. The build stage no longer prescribes ordering, tagging, or decomposition.
- **`spec-guide.md`** shrunk. The no-"and" split rule became a judgment call (split only on independent value or risk); the walking-skeleton mandate and the embedded Claude Design prompt are gone.
- **Standards** are now applied proportionate to what a feature touches, rather than defaulted on wholesale.

## Dropped

- The **walking-skeleton mandate** — a skeleton is now a legitimate *choice* for a project with real seams, never a requirement.
- **`@frozen`/`@scaffolding` test tagging**, the **test-first ordering** rule, and the **coverage map** appended to `spec.md`.
- **`build-deviations.md`** and the **`## Adversarial gate`** record — divergences and risks are reported in the PR, which was always the real channel.
- The **"preserve three independent contexts" doctrine** — independence is retained only where it earns its cost (the risk review), not as a structural rule over the whole flow.

## What to watch in practice

- **Acceptance that nobody checks.** The redesign only holds if every acceptance item maps to a real check — CI, a health probe, an inspectable PR section. An unenforced acceptance item is just procedure you stopped running; drop it or wire it to a check.
- **Risk review sizing.** "Proportional to stakes" is a judgment. If it quietly collapses to "never," the one net under a constitution-blind spec is gone — watch that security-relevant features still draw a real pass.
- **Spec quality upstream.** Intake and the risk review remain the nets. Large `/ship` question batches mean the upstream chat isn't reading `spec-guide.md`/the constitution closely enough.
