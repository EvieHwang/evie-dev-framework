# [Project Name]

[One-line description of what this app is. Replace when this template is copied.]

## Repo map
- `declaration.md` — what this project is and why it exists
- `constitution.md` — principles, standards, and decisions that govern implementation
- `features/[name]/` — per-feature artifacts (intent / plan / verify for Tier 2; the full set for Tier 3)

## Tier detection
The build agent picks a tier from the artifacts present in the feature folder:

- `features/[name]/dag.md` exists       → Tier 3
- `features/[name]/intent.md` exists    → Tier 2
- Neither                               → Tier 1

Before any build action: read `constitution.md`.
Before any architectural decision: read `constitution.md` and `declaration.md`.

## Development environment
Development runs in Claude Code cloud sandboxes attached to this GitHub repo.

- The container is ephemeral and re-cloned each session. Anything not committed and pushed is lost.
- No `~/.claude/CLAUDE.md` exists in the sandbox — user-global preferences are carried at the bottom of this file.
- GitHub access is via the GitHub MCP server (tools prefixed `mcp__github__`). The `gh` CLI is not available.
- Development branch pattern: `claude/<short-task-name>-<suffix>`. Open a PR to `main` when work is complete.

## Run, test, deps
[Fill in when this template is copied:]
- Install: `…`
- Run locally: `…`
- Tests: `…`
- Package manager / lockfile: `…`

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
