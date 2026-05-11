# Global preferences — Evie Hwang

## GitHub Credentials
- GitHub account: use EvieHwang (personal) for public AWS apps, and  Eve-Hwang (organization) for private Eviebot-based apps.
- Self-hosted runner: Eviebot (org-level, Default runner group)
- Runner deploys via launchctl on macOS

## AWS Credentials
- AWS account: 070840362692 (user: eve-hwang) — used for public-facing production apps
- Default AWS region: us-east-1 (confirm per project)
- AWS credentials: use named profiles, never hardcode keys

## Secrets Pattern
- Secrets are stored in GitHub Actions Secrets at the Eve-Hwang org level
- Deployment workflow writes a `.env` file to `~/services/<repo-name>/.env` on Eviebot during deploy
- Always provide a `.env.example` with all required keys (no values) committed to the repo
- Never hardcode secrets in code or commit `.env` files
- Secrets on Eviebot are artifacts of deployment — if Eviebot is rebuilt, the next deploy recreates them
- To add a new secret: add to Eve-Hwang org secrets in GitHub, add the key to `.env.example`, add the inject step to the deployment workflow

## Eviebot — Runtime Server
I have a Mac Mini called Eviebot that serves as my personal server and deployment target. 
- OS: macOS (Mac Mini) — use launchd, not systemd
- Python: python3 — confirm version with `python3 --version` before assuming 3.12
- Services directory: /Users/eviebot/services/<repo-name>/ — directory must exist before first deploy; create manually or with mkdir -p in the workflow
- Use a virtualenv at ~/services/<repo-name>/.venv
- Runner user: eviebot — all paths are /Users/eviebot/
- System Python: Use python3.11 (Homebrew) for anything requiring 3.10+
- Secrets: stored in ~/.secrets/<service-name>.env on Eviebot; never committed


## Deployment Workflow
- Open a PR to main when work is complete
- Commit after each logical unit of work using conventional commits
- Feature files are never modified during implementation — they are the spec
- Prefer self-hosted runners on Eviebot over SSH-based deploy steps
- All deployments run through GitHub Actions triggered on push to main
- All services follow the fastmail-mcp-v2 pattern: pyproject.toml with a console script entry point (not requirements.txt), plists in deploy/ (not launchd/)
- Deploy workflow: rsync to /Users/eviebot/services/<repo-name>/, create venv, pip install, write .env from secrets, kickstart service
- launchctl bootstrap fails from the runner (non-Aqua session) — first load must be done manually from a terminal on Eviebot: launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/<plist>
- All subsequent deploys use launchctl kickstart -k gui/$(id -u)/<label>

## Defaults I'll Override Explicitly
- Always create a README and a CLAUDE.md on repo init — keep it current
- No Docker for local development unless the project has multi-service dependencies
- Frontend: React + TypeScript + Tailwind + shadcn/ui; vanilla JS only for stateless single-page tools
- Layout decisions: apply Nielsen's 10 heuristics — function over decoration
- Aesthetic: information-dense, 14px base, tight line heights, dark mode first-class

## Gateway Integration
- Gateway: eviebot-mcp-gateway repo, running on port 8080
- To add a new MCP backend: follow the steps in the gateway repo's CLAUDE.md exactly before building
- Port allocation, auth patterns (A/B), and the gateway.py block are defined there — read it first
- Existing service labels: check launchctl list | grep eviebot before choosing a label to avoid conflicts with running services
