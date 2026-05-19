# [Project Name]

[One-line description of what this app is. Replace when this template is copied.]

## Repo map
- `declaration.md` — what this project is and why it exists
- `constitution.md` — principles, standards, decisions, and project conventions (testing framework, state-file format, etc.)
- `features/[feature-name]-[number]/` — per-feature artifacts, produced by tier-specific skills at runtime

Before any build action: read `constitution.md`.
Before any architectural decision: read `constitution.md` and `declaration.md`.

## Development environment
Development runs in Claude Code cloud sandboxes attached to this GitHub repo.

- The container is ephemeral and re-cloned each session. Anything not committed and pushed is lost.
- No `~/.claude/CLAUDE.md` exists in the sandbox — user-global preferences are carried at the bottom of this file.
- GitHub access is via the GitHub MCP server (tools prefixed `mcp__github__`). The `gh` CLI is not available.
- Development branch pattern: `claude/<short-task-name>-<suffix>`. Open a PR to `main` when work is complete. Always assign the PR to the repo owner. 

## Run, test, deps
Pick the block that matches your project's stack. Uncomment it and delete the others. The header on each block describes when to use it.

<!--
### Python service (a backend program or MCP server, typically deployed to Eviebot)
- Install: `python3.11 -m venv .venv && .venv/bin/pip install -e .`
  Creates an isolated Python environment in `.venv/` (a hidden folder) and installs this project into it. The `-e` flag means "editable" — code changes take effect without reinstalling.
- Run locally: `.venv/bin/<script-name>` where `<script-name>` is defined in `pyproject.toml` under `[project.scripts]`.
- Tests: `.venv/bin/pytest` — pytest is the standard Python testing framework.
- Package manager / lockfile: `pyproject.toml` describes dependencies. To add one, edit `pyproject.toml` and re-run the install command.
-->

<!--
### Web frontend (React + Vite + Tailwind + shadcn/ui — the constitution's default frontend stack)
- Install: `pnpm install`
  pnpm is a JavaScript package manager (an alternative to npm). Installs everything listed in `package.json`.
- Run locally: `pnpm dev` — starts the Vite dev server. Open http://localhost:5173 in a browser.
- Tests: `pnpm test` — runs Vitest (the test runner that pairs with Vite).
- Package manager / lockfile: `pnpm` with `pnpm-lock.yaml`. To add a dependency: `pnpm add <package-name>`.
-->

<!--
### Next.js web app (React with built-in routing and server-side rendering)
- Install: `pnpm install`
- Run locally: `pnpm dev` — opens http://localhost:3000.
- Tests: `pnpm test`
- Package manager / lockfile: `pnpm` with `pnpm-lock.yaml`. To add a dependency: `pnpm add <package-name>`.
-->

<!--
### iOS or macOS app (Xcode + Swift)
- Install: open `<ProjectName>.xcodeproj` in Xcode. Swift Package Manager (SwiftPM) resolves dependencies automatically on open. No separate install command.
- Run locally: in Xcode, press Run (⌘R) to launch in the chosen simulator or device.
- Tests: in Xcode, press Test (⌘U). Or from the command line:
  `xcodebuild test -scheme <SchemeName> -destination 'platform=iOS Simulator,name=iPhone 15'`
- Package manager / lockfile: Swift Package Manager (`Package.resolved` — committed automatically by Xcode).
-->


## Deployment target
Pick one. Uncomment the matching block and delete the others.

<!--
### Eviebot (headless always-on Mac mini)
- Service path: `/Users/eviebot/services/<repo-name>/` (must exist before first deploy)
- Venv: `.venv/` inside the service path
- Process management: `launchd`; plists live in `deploy/`
- First load is manual on Eviebot (the runner's non-Aqua session can't bootstrap):
  `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/<plist>`
- Subsequent deploys: `launchctl kickstart -k gui/$(id -u)/<label>`
- Deploy via GitHub Actions on push to `main`, on the Eviebot self-hosted runner.
  Workflow: rsync → create venv → pip install → write `.env` from secrets → kickstart.
- Before choosing a launchctl label: `launchctl list | grep eviebot` to avoid collisions.
-->

<!--
### AWS (public-facing)
- Account: `070840362692` (user: `eve-hwang`)
- Region: `us-east-1` (confirm per project)
- Credentials: named profiles only; never hardcode keys
- Deploy via GitHub Actions on push to `main`
-->

<!--
### Apple platform (iOS / macOS via the Xcode Claude extension)
- Specs live in this repo; build runs in Xcode locally
- No CI deploy path; release is a manual Xcode build and submission
-->

## Secrets
- Canonical source: **GitHub Actions Secrets at the Eve-Hwang org level.**
- Commit a `.env.example` listing every required key with no values.
- Never commit `.env` or any file containing secret values.
- On deploy, the workflow injects secrets into the chosen target (writes `.env` on Eviebot, sets env vars on AWS, configures the Xcode build). Anything on the target is an artifact of deployment, not a source of truth — if the target is rebuilt, the next deploy recreates it.
- Adding a new secret: add to Eve-Hwang org secrets → add the key to `.env.example` → add the inject step to the deploy workflow.

---

## User globals — Evie Hwang
*Carried in this file because Claude Code cloud sandboxes have no `~/.claude/CLAUDE.md`. These coordinates apply to every project, not just this one.*

### GitHub
- `EvieHwang` (personal) — used for public AWS-hosted apps
- `Eve-Hwang` (organization) — used for private Eviebot-hosted apps
- Self-hosted runner: **Eviebot** (org-level, Default runner group)

### AWS
- Account: `070840362692` (user: `eve-hwang`)
- Default region: `us-east-1`

### Eviebot — runtime server
- Mac mini, headless, always-on. macOS — use `launchd`, not systemd.
- Runner user: `eviebot`. All paths under `/Users/eviebot/`.
- Services live at `/Users/eviebot/services/<repo-name>/` with a venv at `.venv/`.
- Use `python3.11` (Homebrew) when 3.10+ is needed; otherwise confirm `python3 --version` before assuming.

### Gateway integration
- Gateway repo: `eviebot-mcp-gateway`, running on port 8080.
- To add a new MCP backend, follow the gateway repo's `CLAUDE.md` exactly. Port allocation, auth patterns (A/B), and the `gateway.py` block are defined there — read it first.
- Check existing service labels with `launchctl list | grep eviebot` before choosing a new one.
