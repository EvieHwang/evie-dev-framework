# Tiered Agentic Build Framework
## User Guide

---

## Why This Exists

The bottleneck in AI-assisted development is not generation. Models can write code quickly. The problem is that without structure, they write the wrong code quickly — optimizing toward plausible output rather than toward your actual intent. Most AI-assisted workflows collapse because there is no enforced contract between what you wanted, what you specified, and what got built.

This framework makes that contract explicit and executable. Every piece of work passes through a pre-build sequence that converts intent into artifacts, and artifacts into a build that runs until tests pass. The human is most present at the front, where judgment matters most. The agent runs autonomously at the back, where mechanical precision matters most.

The framework operates at three tiers. Choosing the right tier for the right work is the first and most important decision you make.

---

## The Three Tiers

### Tier 1 — Bug Fix

For bounded, well-understood problems where the intent is given by the bug itself. The structure already exists. The correct behavior is already known. The only open question is where the problem is and what the minimal fix looks like.

Tier 1 uses Claude Code's built-in plan mode and build mode. No pre-build artifacts are generated. A single session reads the repo, proposes a fix, implements it, runs existing tests, and opens a PR.

**Choose Tier 1 when:** something is broken and you know what correct looks like.

**Do not choose Tier 1 when:** the scope is unclear, the fix touches multiple components, or the correct behavior is itself in question.

---

### Tier 2 — Feature Build

For real but bounded features where the intent needs to be formed before building begins. Three skills run in sequence, each producing a document. Those documents load into a single context window build session that runs until tests pass.

The pre-build sequence is: Intent → Plan → Verify → Build.

**Choose Tier 2 when:** you are adding a meaningful feature to an existing system and could complete it in a focused session. The work is deliberate but contained.

**Do not choose Tier 2 when:** the feature is large enough that it cannot reasonably be held in a single context window, or when it involves multiple significant architectural decisions that need their own feedback loops.

---

### Tier 3 — Full System Build

For large features or full applications where the work is too complex to hold in a single context window. A full pre-build sequence produces a set of committed artifacts before any code is written. The build is driven by a dependency graph that advances across multiple sessions, with tests running at every step.

**Choose Tier 3 when:** you are building something that spans multiple components, requires architectural decisions that will be hard to reverse, or will take more than a single focused session to complete.

**Do not choose Tier 3 when:** Tier 2 will do. The overhead of a full pre-build sequence is recovered quickly on large work. On small work it is waste.

---

## The Repo Structure

Every project uses the same repo shape. The tier is not a setting — it is implicit in which artifacts have been populated.

```
/
├── CLAUDE.md
├── declaration.md
├── constitution.md
│
├── features/
│   └── [feature-name]/
│       ├── intent.md              # T2
│       ├── plan.md                # T2
│       ├── verify.md              # T2
│       ├── requirements.md        # T3
│       ├── design.md              # T3
│       ├── research.md            # T3
│       ├── adversarial-review.md  # T3
│       ├── dag.md                 # T3
│       ├── state.md               # T3
│       └── tests/
│           └── *.feature
│
└── .claude/
    ├── settings.json
    └── commands/
        ├── t1-build.md
        ├── t2-intent.md
        ├── t2-plan.md
        ├── t2-verify.md
        ├── t2-build.md
        ├── t3-declaration.md
        ├── t3-research.md
        ├── t3-requirements.md
        ├── t3-architecture.md
        ├── t3-adversarial.md
        ├── t3-test-coach.md
        ├── t3-generate-dag.md
        ├── t3-next-step.md
        └── t3-build.md
```

Declaration and constitution live at the root because they are project-level. Everything else scopes to the feature folder. Each feature carries its own tier implicitly — if `dag.md` exists in the feature folder, it is a Tier 3 feature. If only `intent.md` exists, it is Tier 2.

The `.claude/commands/` folder contains all skills for all tiers. You choose which to invoke at runtime. Nothing enforces a tier — the choice is yours at the start of each piece of work.

---

## The Core Documents

Three files must exist in every project before any build work begins. These are the only files that are project-level rather than feature-level.

### CLAUDE.md

The entry point. Claude Code reads this automatically on startup. Its job is operational context and navigation — not principles, not intent. Keep it thin.

CLAUDE.md should contain: how to run the project locally, where dependencies are managed, how to run tests, where auth and secrets live (never commit these), a brief description of the repo layout, and pointers to constitution.md and declaration.md.

It should also contain tier detection logic so the build agent orients correctly:

```
## Tier detection
- features/[name]/dag.md exists → Tier 3
- features/[name]/intent.md exists → Tier 2
- Neither → Tier 1

Before any build action: read constitution.md.
Before any architectural decision: read constitution.md 
and declaration.md.
```

### declaration.md

The originating intent for the project as a whole. What it is, why it exists, for whom, and what it explicitly does not do. Written once, at the start, and held stable. If you find yourself revising the declaration during a build, it usually means the declaration contained implementation assumptions that were really requirements.

A declaration is written at the right level of abstraction when it describes what you want to exist and why, without specifying how. Everything else goes elsewhere.

### constitution.md

The accumulated judgment of the project. It contains the decisions that have already been made and should not be re-litigated, the principles that govern how implementation decisions get made, the standards the project follows, and the quality gates that must be met before anything ships.

The constitution is not a per-feature document. It is the persistent frame that makes the project coherent across sessions, features, and contributors. An agent that has read the constitution should never make a structural decision that contradicts it without surfacing that as an explicit choice.

The constitution grows over time. Each significant decision that gets made should be logged in the decision log with its rationale. This is what prevents future agents from relitigating settled questions.

---

## The Standards Registry

The constitution contains a standards registry — a set of named external authorities that govern design and implementation decisions. These are standards that already exist, are well-documented, and that Claude follows from a name or URL without requiring further elaboration.

The default registry covers the vast majority of projects:

**Interface:** Apple Human Interface Guidelines. Applies to all interface decisions, not only iOS surfaces. Covers interaction patterns, navigation, affordances, and visual hierarchy.
`developer.apple.com/design/human-interface-guidelines`

**Accessibility:** WCAG 2.1 AA. The baseline for all user-facing surfaces. Non-negotiable for civic-facing work.
`w3.org/WAI/WCAG21/quickref`

**Security:** OWASP Top 10. The most critical web application security risks. Establishes a security mindset across all implementation decisions.
`owasp.org/www-project-top-ten`

**API contracts:** OpenAPI Specification. How interfaces between components get described whenever services communicate.
`spec.openapis.org`

Add from the extended set when relevant:

**API design:** Microsoft REST API Guidelines — when building APIs for external consumers.
**AI integration:** OWASP Top 10 for LLMs — when building AI-integrated applications.
**Security depth:** OWASP ASVS — when handling sensitive data or building for high-assurance contexts.

The rule for the standards registry: agents follow these without deviation unless the declaration explicitly requires otherwise. Deviation must be surfaced as a decision, not made silently.

---

## Running Tier 1

Tier 1 requires no pre-build work. Open Claude Code and run:

```
/project:t1-build
```

The agent reads CLAUDE.md and constitution.md, identifies the affected module and any relevant specs, proposes a minimal fix, shows you the plan before implementing, runs existing tests on completion, and opens a PR describing what was broken, what changed, and what was verified.

The one discipline to maintain: before approving the plan, ask what the fix could break. The agent will surface this if the t1-build command includes it, but it is worth making a habit regardless.

---

## Running Tier 2

Tier 2 runs four commands in sequence. The first three produce documents. The fourth runs the build.

### Step 1 — Intent

```
/project:t2-intent feature-name: [name]
```

The skill runs a coached conversation first. It asks questions to draw out what you are building, why it exists, who needs it, and what success looks like from their perspective. Answer fully. The quality of everything downstream depends on the quality of this conversation.

When intent is clearly formed, confirm and the skill moves to its second phase — converting the conversation into a structured intent document. This document contains the purpose statement, user stories, and explicit out-of-scope boundaries.

You are the only human gate at this stage. Do not confirm until the intent document says what you actually meant.

### Step 2 — Plan

```
/project:t2-plan feature-name: [name]
```

The plan skill reads the intent document and works in two phases. First it identifies the architectural considerations that the intent implies — what components are affected, what constraints apply, what decisions must be made before implementation begins. Then it derives the implementation approach from those constraints.

The plan does not restate intent. It answers how, given the what that intent established. If the plan surfaces architectural considerations that reshape what you thought you wanted, update the intent document before proceeding.

### Step 3 — Verify

```
/project:t2-verify feature-name: [name]
```

The verify skill reads intent first, then plan, in that order. It derives two categories of tests: behavioral tests from intent (does it do what the user needs?) and structural tests from plan (does it implement correctly given the constraints?). Tests are written in Gherkin format and output as executable feature files.

If the verify skill surfaces a requirement it cannot test as written, that is a signal the requirement is underspecified. Resolve it before building. A weak test is worse than no test because it creates false confidence.

### Step 4 — Build

```
/project:t2-build feature-name: [name]
```

The build agent reads all three documents in order — constitution, intent, plan, verify — and builds until all tests pass. It does not consider the feature complete until every test passes. On a test failure it attempts one remediation; if the test fails again it stops and reports rather than proceeding.

On completion it opens a PR referencing the intent document, the plan, and the test results.

You do not need to be present during the build. The documents carry the intent. The tests carry the exit condition.

---

## Running Tier 3

Tier 3 runs a full pre-build sequence before any code is written. Human judgment is concentrated at the front. The build runs autonomously.

### The Pre-Build Sequence

**Declaration** — if the project declaration does not exist or needs updating for this scope of work, run `/project:t3-declaration`. This is a coached conversation that produces or refines declaration.md.

**Requirements v1** — run `/project:t3-requirements`. Produces a hypothesis-level requirements document. Not a commitment — a starting point for architectural review.

**Sketch architecture** — run `/project:t3-architecture`. The agent identifies components, contracts, and constraints, then flags any requirements that the architecture implies should change. Update requirements.md accordingly. This feedback loop between architecture and requirements is the most important quality lever in Tier 3. Invest in it.

**Research** — run `/project:t3-research` if the work involves external dependencies, existing tools, or a landscape you have not fully mapped. Research findings feed back into requirements and architecture.

**Requirements v2** — revise requirements.md with everything learned from architecture and research. This is the commit point. Requirements should not change after this without a deliberate decision.

**Full architecture** — the architecture skill runs again to produce the committed design given refined requirements. This is what the DAG will be built from.

**Adversarial review** — run `/project:t3-adversarial`. The agent is explicitly tasked with finding problems. Zero findings is a failure condition. Surface anything it flags before proceeding to DAG generation.

**DAG generation** — run `/project:t3-generate-dag`. The agent produces a dependency graph of all build tasks, organized into parallel waves. A hook fires automatically that runs the test-coach skill to generate the test suite.

### The Build Loop

Once the DAG and test suite exist, the build runs with minimal human involvement.

```
/project:t3-build feature-name: [name]
```

The build agent reads the declaration, constitution, DAG, state, and tests. It dispatches independent tasks within each wave as parallel sub-agents. After each wave completes it runs the full test suite. On a test failure it stops and reports — it does not advance until the failure is resolved.

State is updated after every step. If a session ends before the DAG is complete, the next session reads state and picks up exactly where it stopped.

When the DAG is complete and all tests pass, the agent opens a PR with the DAG completion summary and wave-by-wave test results.

### Where the Human Gates Are

Declaration confirmation. Architecture review — specifically the feedback it surfaces into requirements. Adversarial review findings — you decide whether each finding requires a change before the DAG runs. Test failures during the build loop — these require your judgment if the agent's remediation attempt fails.

Everything else runs. That is the design.

---

## Feedback Loops and Iteration

Iteration is expected and is not failure. The pre-build sequence is designed to surface the right feedback at the right stage rather than discovering it during implementation.

The most common feedback loop is architecture back to requirements. Architecture surfaces constraints that make certain requirements impossible, expensive, or that reveal requirements you did not know you had. This is why requirements v1 is explicitly a hypothesis. The revision after architectural review is not rework — it is the sequence working correctly.

The second most common is test definition back to requirements. Writing executable tests exposes ambiguity that narrative requirements obscure. If the verify or test-coach skill cannot write a clean test for a requirement, the requirement is not specified well enough.

The adversarial review catches a class of problems that neither architecture nor testing surfaces — contradictions between documents, scope drift, silent failure modes. It should be the last thing before the DAG runs and the findings should be taken seriously.

If the build loop surfaces failures, trace them back before fixing forward. A test failure during build usually means something was underspecified in the plan or requirements. Fix the document, understand why it was underspecified, and update the constitution if the gap reveals a principle that should be standing guidance.

Over time this learning loop is how the constitution grows. Each failure that traces back to a missing principle becomes a new constitution entry. The constitution becomes more complete; the pipeline becomes more reliable.

---

## Prompt Discipline

The documents you write — declaration, constitution, intent, plan — are prompts. Their quality determines the quality of what the agent produces. The principles that make a good system prompt make a good artifact.

**Positive instructions outperform negative ones.** "Respond in plain prose" is stronger than "do not use bullet points." State what you want; avoid cataloguing what you do not want unless the default behavior is specifically what you are overriding.

**Signal density outperforms length.** The model does not reward thoroughness — it rewards precision. A constitution entry that says one thing clearly is more effective than three paragraphs that say the same thing from different angles. Restatement dilutes rather than reinforces.

**Examples outperform explanation for behavior and style.** One well-chosen example does more work than a paragraph of description. Use examples when the desired output is stylistic or when the behavior is genuinely ambiguous without one.

**Negative instructions have a specific use.** They are worth writing when the behavior you are prohibiting is what the model would do by default. "Do not advance past a failing test" overrides a default. "Do not be unhelpful" does not override anything and costs tokens.

**Exit conditions are underused and high value.** Telling the agent exactly what done looks like removes a large class of ambiguity that otherwise gets resolved arbitrarily. Every build command should have a clear exit condition.

The same discipline applies to the coached conversations in the intent and declaration skills. Direct, complete, no hedging. The coaching works best when you say what you actually mean rather than what you think the process wants to hear.

---

## New Project Setup

### Before you start

You need the template repo cloned or forked. All the `.claude/commands/` files should be in place. The `features/` directory should exist with a README explaining its purpose.

### Step 1 — Write constitution.md

Start here, not with declaration. The constitution establishes the constraints that everything else must respect. Use the template from the repo and fill in:

The standards registry — the default set covers most projects. Add from the extended set if you know you will need API design standards, LLM security, or deeper security verification.

Architectural principles — if you already know structural rules that apply (technology choices you have committed to, patterns you are following, things that are out of scope for the project entirely), write them now. Leave the decision log empty — it will populate as you build.

Quality gates — what must be true before any PR. At minimum: all tests pass. Add anything specific to your project — accessibility compliance, security scan, code review.

### Step 2 — Write declaration.md

Answer three questions:
- What is this? One or two sentences, no implementation detail.
- Why does it exist? The problem it solves or the need it serves.
- For whom? Who uses it and what do they need from it.

Add an explicit out-of-scope section. This is one of the most valuable things in the document — it prevents scope creep from being encoded silently into implementation decisions.

Hold this document at the right level of abstraction. If you find yourself writing how it works rather than what it is, move that content to constitution or leave it for requirements.

### Step 3 — Write CLAUDE.md

Fill in the operational context: how to run the project, where dependencies are managed, how to run tests, where auth and secrets live. Add the tier detection logic. Point to constitution.md and declaration.md explicitly.

Keep it thin. Anything that is a principle or a decision belongs in the constitution. Anything that is intent belongs in the declaration. CLAUDE.md is a map, not a document.

### Step 4 — Create your first feature folder

```
features/[feature-name]/
└── README.md    "Feature artifacts live here. 
                  Populated by skills at runtime."
```

You are now ready to run a tier.

### Step 5 — Choose your tier and run

**Tier 1:** open Claude Code, describe the bug, run `/project:t1-build`.

**Tier 2:** run `/project:t2-intent feature-name: [name]` and work through the conversation. Then plan, verify, build in sequence.

**Tier 3:** begin with `/project:t3-declaration` if the project declaration needs work, then run the full pre-build sequence in order.

---

## Quick Reference

| | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|
| Use for | Bug fix | Feature build | Full app or large feature |
| Sessions | Single | Single | Multi |
| Pre-build | None | Intent, Plan, Verify | Full sequence |
| Human gates | Plan review | Intent confirmation | Declaration, architecture, adversarial review |
| Build | Plan + build mode | Single context window | DAG-driven multi-session |
| Exit condition | Existing tests pass | All verify tests pass | All DAG steps complete, all tests pass |
| PR | Yes | Yes | Yes |

### Commands
```
/project:t1-build
/project:t2-intent    feature-name: [name]
/project:t2-plan      feature-name: [name]
/project:t2-verify    feature-name: [name]
/project:t2-build     feature-name: [name]
/project:t3-declaration
/project:t3-research      feature-name: [name]
/project:t3-requirements  feature-name: [name]
/project:t3-architecture  feature-name: [name]
/project:t3-adversarial   feature-name: [name]
/project:t3-test-coach    feature-name: [name]
/project:t3-generate-dag  feature-name: [name]
/project:t3-next-step     feature-name: [name]
/project:t3-build         feature-name: [name]
```

### Default standards registry
```
Interface:     Apple Human Interface Guidelines
Accessibility: WCAG 2.1 AA
Security:      OWASP Top 10
API contracts: OpenAPI Specification
```

---

*This framework is a living document. When the build loop surfaces something the constitution did not anticipate, update the constitution. When a tier boundary proves wrong in practice, revise it. The framework should reflect how you actually work, not how you planned to work.*
