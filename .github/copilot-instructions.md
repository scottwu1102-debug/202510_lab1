<!--
This file guides AI coding agents (Copilot-style) to be productive in this repository.
Keep it short, concrete and focused on discoverable patterns in the codebase.
-->
# Copilot instructions for 202510_lab1

Overview
- Purpose: small static web app (tic-tac-toe) served by Nginx. The repository contains a Dockerfile, docker-compose snippet, a GitHub Actions security pipeline, and a simple SPA in `app/`.
- Runtime: static front-end only. No backend services in this repo.

What to change and why
- UI/logic lives in `app/script.js`, markup in `app/index.html`, and styles in `app/style.css`. Small changes to UX, AI opponent, or scores should be done here.
- Containerization and deployment are handled by `Dockerfile`, `docker-compose.yml` (project-level snippet) and the workflow at `.github/workflows/secure-pipeline.yml`.

Developer workflows (how to build, run, test)
- Local quick test: open `app/index.html` in a browser or run the Docker image exposed on port 8080.
- Build + run with docker-compose (project uses image tag ghcr.io/tryweb/202510_lab1:latest in compose): the Dockerfile builds an nginx image that copies `app/` into `/usr/share/nginx/html` and listens on port 8080.
- CI: `.github/workflows/secure-pipeline.yml` runs Semgrep, optional dependency scans, Trivy for container images, Checkov for IaC, and gitleaks for secrets. Avoid changing pipeline semantics unless security implications are explicit.

Project-specific conventions and important details
- Port: Nginx is configured to listen on 8080 (see `Dockerfile` and `nginx.conf`). Expose changes must keep that port mapping.
- Static-only: there is no server-side code here. Any change that needs runtime secrets or server logic must add new services and corresponding CI updates.
- Security-aware repo: CI explicitly expects static web app; many CI steps conditionally skip scans when no matching files are present. New languages/dependencies should include conventional lockfiles (package.json, requirements.txt, etc.) so SCA steps trigger.

Notable code patterns to be careful about
- `app/script.js` intentionally contains insecure examples (for educational/demo purposes): unsafe uses of `eval`, `setTimeout` with string, innerHTML assignment, hard-coded secrets (API_KEY, DATABASE_URL), and a risky regex. When fixing, prefer minimal, well-scoped changes and preserve demo intent unless instructed otherwise.
  - Examples:
    - `evaluateUserInput(input) { return eval(input); }` — replace with a safe parser or remove if the demo requires showing CWE examples.
    - `setTimeout('computerMove()', userInput)` — replace with `setTimeout(computerMove, Number(userInput) || default)`.
    - `statusDisplay.innerHTML = '<span>' + ...` — use `textContent` or create elements to avoid XSS.

Files to inspect for common edits
- UI/behavior: `app/script.js`, `app/index.html`, `app/style.css`
- Container and runtime: `Dockerfile`, `nginx.conf`, `docker-compose.yml`
- CI/security: `.github/workflows/secure-pipeline.yml`

Editing rules for AI agents
- Make minimal, single-purpose commits. Each PR should: modify only UI/logic files for gameplay changes; modify Dockerfile/nginx.conf only for packaging/runtime fixes; modify workflow only to add well-justified automation steps.
- Preserve existing file structure. When adding new languages or tooling, include standard lockfiles so CI SCA detectors trigger.
- Do not commit real secrets. The repo contains deliberate example secrets — replace with environment variables and document the change in the PR description.

When unsure, where to look
- For UI behaviour look at `app/script.js` (game logic, Minimax, scoring). For layout and static assets `app/index.html` and `app/style.css`.
- For how the app is hosted look at `Dockerfile` and `nginx.conf` (notice listen 8080, removal of `user nginx` and tmp pid workaround).
- For security expectations and CI checks inspect `.github/workflows/secure-pipeline.yml`.

What I cannot infer
- No backend service contracts, tests, or package manifests exist. If you need to add server-side components or test frameworks, update the CI workflow and add dependency manifests (e.g. `package.json`) so scans run correctly.

If you modify this file
- Keep it short. Reference actual files (paths above). Avoid speculative process changes.

Request for feedback
- If any sections are unclear or you want more automation examples (local Docker run commands, test harness), tell me which area to expand.
