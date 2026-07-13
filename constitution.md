# Constitution

## Standards
Interface:        Apple Human Interface Guidelines
                  developer.apple.com/design/human-interface-guidelines
Accessibility:    WCAG 2.1 AA
                  w3.org/WAI/WCAG21/quickref
Security:         OWASP Top 10
                  owasp.org/www-project-top-ten
API contracts:    OpenAPI Specification
                  spec.openapis.org

Extended (apply when relevant):
API design:       Microsoft REST API Guidelines
AI integration:   OWASP Top 10 for LLMs
Security depth:   OWASP ASVS

These apply where the project's platform and exposure make 
them relevant, honored as acceptance criteria proportionate to 
what the feature actually touches — a one-page admin tool does 
not absorb a full accessibility sign-off. A deviation, or loading 
a heavy cross-cutting standard onto a thin feature, is surfaced 
as a decision, not made silently or absorbed as a hidden cost.

## Architectural principles
- The spec is the contract. If a requirement turns out to be wrong, it is surfaced to the owner, not silently patched — the party trying to meet the definition of done does not get to edit it (see `## Acceptance bar`).
- No Docker for local development unless the project has multi-service dependencies that genuinely require it.
- All deployments run through GitHub Actions, triggered by push to `main`.
- Prefer first-class deploy tooling (`flyctl deploy` from GitHub Actions) over SSH-based deploy steps.
- A deploy is done when the release is **verified healthy** — a real probe of the running service — not when CI turns green.

[Add app-specific principles below as they are decided.]

## Acceptance bar (definition of done)
*What `/ship` must be able to demonstrate for a feature to be done. It specifies outputs, not the path — how the build reaches them is the builder's business. Every item here is something a person or CI can actually check; an item that can't be checked doesn't belong on the bar.*

- **Behavior holds.** The spec's behavioral requirements are satisfied and covered by tests that verify them as written, and those tests are green — **every** runner in `## Testing`, not just one.
- **The gates pass.** Everything in `## Quality gates` is met.
- **Standards honored where they apply** (`## Standards`), proportionate to what the feature touches.
- **The release is verified healthy** — a real probe of the running service, not merely green CI (see the architectural principles).
- **Spec integrity preserved.** A requirement that looks *wrong* is surfaced to the owner, never silently patched. Where the build had to diverge from the spec's *design* (not its requirements) to satisfy the behavior, that divergence is reported in the PR so it can feed `## Spec-authoring lessons`.

How the tests get written — in what order, against which internal surfaces, before or after the code — is not the framework's concern. The bar is that the behavior is real, checked, and green. A weak test that creates false confidence, and a premature test that pins internal shape and forecloses valid implementations, are both defects; neither is fixed by committing the other.

## Patterns in use
- **Frontend stack:** React + TypeScript + Tailwind + shadcn/ui. Vanilla JS is acceptable only for stateless single-page tools.
- **UI aesthetic:** information-dense, 14px base, tight line heights, dark mode as a first-class surface.
- **UX checklist:** apply Nielsen's 10 heuristics as a general sanity-check on any interface; Apple HIG (see Standards) is the authoritative reference for platform decisions.
- **Python service layout:** `pyproject.toml` with a console-script entry point (not `requirements.txt`); Fly deployment config (`fly.toml`, and a `Dockerfile` if the build needs one) lives at the repo root.

[Add app-specific patterns below as they are established.]

## Spec-authoring lessons
*Recurring spec/test-authoring mistakes learned from real builds, surfaced by `/ship` from the design divergences it reports in the PR, so they don't recur. Both the upstream spec author (per `spec-guide.md`) and `/ship` read this section before working. Keep each entry concrete: the mistake, and the rule that prevents it, tagged with the feature it came from.*

- [e.g. "Don't assert the exact JWT algorithm in a test — assert that an expired/forged token is rejected. (auth-1)"]

## Quality gates
- All tests pass — and CI runs the same build the deploy runs. If the test runner doesn't type-check/compile (Vitest, esbuild, isolatedModules), CI also runs the production build (`tsc` / `pnpm build` / `mypy` / `go build`).
- `README.md` and `CLAUDE.md` exist and are current.
- `.env.example` lists every key the deploy workflow injects — no drift between the committed example and the actual required secrets.

[Add app-specific gates below as they are established.]

## Testing
*One block per test runner. A polyglot repo (e.g. a Python API plus a JS/TS frontend) has more than one — list each, because the green bar `/ship` must hit is **every** runner passing. `Test root` records how this runner discovers the per-feature suites under `features/*/tests/` — the one-time wiring `/ship` establishes so feature tests are executable without per-feature path hacks. A single-runner project has just one block. Populated by `/ship` on first use.*

- **Runner:** [name, e.g. pytest]
  **Run:** `[command]`
  **Test root:** [how this runner finds feature tests — e.g. a `pytest.ini`/`pyproject` `testpaths`, a Vitest `include` glob, an import alias — or note that feature tests live in `features/*/tests/<runner>/`]

## Out of scope
[List what this codebase explicitly does not do. Populated per app.]

## Decision log
| Date | Decision | Rationale |
|------|----------|-----------|
[populated as significant decisions are made]

## Acknowledged risks
*Cross-feature accumulation surface. Each risk-review finding the owner acknowledges — by an explicit answer during `/ship`, or by merging a PR whose body carries it under "Risks for review" — gets one row here so the project never silently forgets that it knowingly took on risk. Severity is the unmitigated severity — an acknowledged HIGH stays HIGH. Populated by `/ship`.*

| Feature | Finding | Severity | Risk | Rationale | Mitigation |
|---------|---------|----------|------|-----------|------------|
[populated by the independent risk review in /ship]
