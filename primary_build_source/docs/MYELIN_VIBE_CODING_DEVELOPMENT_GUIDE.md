# Myelin - Vibe-Coding Development Guide

## 🚀 Quick Start Checklist

- [ ] Install `Git`.
- [ ] Install `Node.js` LTS and enable `pnpm` with Corepack.
- [ ] Install `Docker Desktop`.
- [ ] Install `FFmpeg`.
- [ ] Install `Cursor` or `VS Code`.
- [ ] Confirm these commands work in a new PowerShell window:
  - `git --version`
  - `node -v`
  - `pnpm -v`
  - `docker --version`
  - `ffmpeg -version`
- [ ] Create a workspace root and keep `primary_build_source/` in it as the source-of-truth docs folder.
- [ ] Bootstrap the `pnpm` + `turbo` monorepo at the workspace root.
- [ ] Add the first three runnable surfaces:
  - `apps/web`
  - `apps/api`
  - `apps/worker`
- [ ] Create local `.env` files with safe development values.
- [ ] Start local Postgres and Redis with Docker.
- [ ] Run:
  - `pnpm install`
  - `pnpm dev`
  - `pnpm lint`
  - `pnpm typecheck`
  - `pnpm test`
  - `pnpm build`

## 📋 Project Overview & Tech Stack

Myelin is an AI-native, simulation-first training platform for engineering trades. The current `primary_build_source` package is not a runnable app yet; it is a build source bundle containing specifications, reference HTML prototypes, design tokens, and implementation decisions for the professional rebuild.

The docs lock Myelin into a browser-first, multi-tenant architecture with a single routed application, a 7-step LOTO simulator, strict demo/live data separation, and AI support that is safety-constrained and context-aware.

### What You Have Today

The current source package contains:

```text
primary_build_source/
  README.md
  docs/
    MYELIN_PROFESSIONAL_REBUILD_PRD.md
    MYELIN_PROFESSIONAL_FEATURES.md
    MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md
    MYELIN_V1_RECOMMENDED_TECH_STACK.md
    MYELIN_AI_AGENT_V1_BUILD_BLUEPRINT.md
    MYELIN_AI_AGENT_IMPLEMENTATION_PLAN.md
    v1_decisions.md
  references/
    product/
    simulation/
    ui_baseline/
  assets/
    styles/myelin-tokens.css
```

### Target Working Repo

This guide assumes you will keep `primary_build_source/` for reference and build the working repo around it:

```text
Myelin Build/
  primary_build_source/
  apps/
    web/
    api/
    worker/
  packages/
    contracts/
    db/
    ui/
    sim-core/
    sim-renderer/
    sim-physics/
    sim-loto/
    sim-telemetry/
  infra/
    docker/
  .vscode/
  package.json
  pnpm-workspace.yaml
  turbo.json
  tsconfig.base.json
```

### Locked Build Rules From The Source Package

These are not optional. Keep them in mind every time you use an AI copilot:

1. Build one routed application. Do not rebuild the prototype as separate standalone module pages.
2. Keep v1 curriculum scope to `FND-TT-01` and `FND-IW-01`.
3. Use the LOTO 7-step simulator as the release-blocking simulation.
4. Keep demo mode isolated from live tenant data.
5. Use `myelin-theme` as the theme storage key.
6. Route post-simulator completion to `/app/submissions/new`, not `assignments`.
7. Use Postgres row-level security for tenant isolation.
8. Keep AI assistance safety-bound: no direct graded answers before submission, no unsafe real-world guidance.

### Tech Stack Summary

| Area | Stack | Source Status |
|---|---|---|
| Monorepo tooling | `pnpm`, `turbo`, TypeScript | Confirmed in `docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md` |
| Frontend | React + Vite + `react-router-dom` + `@tanstack/react-query` | Confirmed |
| Simulator | `three`, `cannon-es`, `gsap`, `idb` | Confirmed |
| Backend | NestJS + Fastify + `kysely` + `pg` + `pino` | Confirmed |
| Queue/worker | `BullMQ` + `ioredis` | Confirmed |
| Data | PostgreSQL + Redis + S3-compatible object storage | Confirmed |
| Testing | Vitest + Playwright | Confirmed |
| Quality tools | ESLint + Prettier | Confirmed |
| Auth/SSO | WorkOS | Confirmed as recommended |
| AI runtime | NVIDIA-first live conversational stack | Confirmed direction, vendor wiring still implementation-specific |

### Entry Points You Should Expect

The source docs imply these entry points for the rebuild:

| Path | Purpose | Status |
|---|---|---|
| `apps/web/src/main.tsx` | React app entry | Assumed from recommended Vite layout |
| `apps/api/src/main.ts` | NestJS API entry | Assumed from recommended layout |
| `apps/worker/src/main.ts` | Worker entry | Assumed from recommended layout |
| `primary_build_source/assets/styles/myelin-tokens.css` | Design tokens reference | Confirmed |
| `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md` | Simulator engineering rules | Confirmed |
| `primary_build_source/docs/v1_decisions.md` | Locked v1 decisions | Confirmed |

## ⚠️ Assumptions Made

The current repo does not include `package.json`, lockfiles, `.env.example`, Dockerfiles, or runnable code. To make this guide usable, these assumptions are explicit:

1. You will use the current workspace root as the implementation repo root and keep `primary_build_source/` inside it.
2. You will use Windows 11 as the primary development machine. macOS and Linux notes are included only where needed.
3. You will use the current Node.js LTS release available when you start. If you need a concrete target, use the current LTS line, not `Current`.
4. You will use `pnpm` as the only package manager. Do not mix `npm`, `yarn`, and `pnpm` lockfiles in this repo.
5. You will create a local Postgres + Redis stack with Docker Desktop for day-one development.
6. You will start with a minimal vertical slice:
   - React app shell
   - NestJS API with `/health`
   - worker process that starts cleanly
7. You will add auth, uploads, AI, and storage integrations later behind feature-ready env vars and scripts.
8. You will use these assumed local ports unless your machine already uses them:
   - web: `3000`
   - api: `4000`
   - postgres: `5432`
   - redis: `6379`
9. You will use these assumed root scripts once the monorepo is scaffolded:
   - `pnpm dev`
   - `pnpm build`
   - `pnpm lint`
   - `pnpm typecheck`
   - `pnpm test`
   - `pnpm test:integration`
   - `pnpm test:sim`
   - `pnpm test:e2e`
   - `pnpm db:migrate`
   - `pnpm db:seed`
10. Production deployment target is not defined in the source package, so this guide assumes a generic container-based stage/prod deployment with managed Postgres, Redis, and S3-compatible storage.

## 🛠️ Phase 1 - Workspace (Zero to Editor)

### 1. Install Git

Search: `Git for Windows download`

Run in PowerShell:

```powershell
git --version
```

Success: you see a version such as `git version 2.x.x`.

If it fails: close and reopen PowerShell after installation, then run the command again.

Git: not applicable yet. This is a machine setup step, not a repo change.

### 2. Install Node.js LTS And Enable pnpm

Search: `Node.js LTS download`

After installation, run in PowerShell:

```powershell
node -v
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v
```

Success:

- `node -v` shows an LTS version.
- `pnpm -v` shows a version instead of an error.

If it fails:

- If `node` is not found, reopen PowerShell.
- If `pnpm` is not found, rerun `corepack enable` and `corepack prepare pnpm@latest --activate`.

Git: not applicable yet.

### 3. Install Docker Desktop

Search: `Docker Desktop Windows download`

After installation, start Docker Desktop and wait until it says it is running. Then run:

```powershell
docker --version
docker info
```

Success:

- `docker --version` returns a version.
- `docker info` returns engine details without an error.

If it fails:

- Make sure Docker Desktop is open.
- If WSL is not configured, follow Docker Desktop's setup wizard and restart the machine if requested.

Git: not applicable yet.

### 4. Install FFmpeg

Search: `FFmpeg Windows build download`

After installation, reopen PowerShell and run:

```powershell
ffmpeg -version
```

Success: FFmpeg prints version and codec information.

If it fails: add the FFmpeg `bin` folder to your Windows `Path`, then reopen PowerShell.

Git: not applicable yet.

### 5. Install Cursor Or VS Code

Search:

- `Cursor editor download`
- `Visual Studio Code download`

Recommended extensions if you use VS Code:

- `dbaeumer.vscode-eslint`
- `esbenp.prettier-vscode`
- `ms-playwright.playwright`
- `ms-azuretools.vscode-docker`
- `eamodio.gitlens`

If you use Cursor, the AI editing experience is built in. You can still install the same linting and Docker extensions.

Success: the editor opens normally and you can open folders from it.

If it fails: reinstall using the default installer options.

Git: not applicable yet.

### 6. Create The Workspace Root

Pick a parent folder such as `C:\Projects` or `C:\Users\<you>\Code`, then create your Myelin folder.

Run in PowerShell:

```powershell
mkdir C:\Projects\myelin
Set-Location C:\Projects\myelin
```

Success: `pwd` shows your new workspace path.

If it fails: choose a folder where your Windows user account has write access.

Git: not applicable yet.

### 7. Add Or Clone The Source Package

Choose one of these paths:

1. If you already have this repo as a zip or copied folder, place `primary_build_source/` inside the workspace root.
2. If you already have a remote repo, clone it into the workspace root and make sure `primary_build_source/` exists at the top level.

If you are starting from the existing source package only, run:

```powershell
git init
git add primary_build_source
git commit -m "chore: add primary build source documents"
```

Success:

- `git status` reports a clean working tree.
- `primary_build_source/` exists at the workspace root.

If it fails:

- If `primary_build_source` is missing, copy it in before committing.
- If Git says author identity is missing, run:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Git:

```powershell
git add primary_build_source
git commit -m "chore: add primary build source documents"
```

### 8. Open The Folder In Cursor Or VS Code

In the editor:

1. Open the workspace root.
2. Click `Trust the authors` or `Trust this workspace` when prompted.
3. Open an integrated terminal in the workspace root.

Success: the editor sidebar shows `primary_build_source/`, and the terminal opens in the same folder.

If it fails: reopen the editor directly from the workspace root folder.

Git: not applicable yet.

### 9. Add Workspace Editor Settings

Create `.vscode/settings.json` with this starter content:

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "files.eol": "\n",
  "typescript.tsdk": "node_modules/typescript/lib",
  "search.exclude": {
    "**/dist": true,
    "**/.turbo": true,
    "**/playwright-report": true,
    "**/test-results": true
  }
}
```

Create `.vscode/extensions.json` with this starter content:

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-playwright.playwright",
    "ms-azuretools.vscode-docker",
    "eamodio.gitlens"
  ]
}
```

Success: both files exist and the editor starts recommending extensions.

If it fails: create the `.vscode` folder manually, then create the two files again.

Git:

```powershell
git add .vscode
git commit -m "chore: add workspace editor settings"
```

### Phase 1 Checklist

- [ ] `git --version` works.
- [ ] `node -v` works.
- [ ] `pnpm -v` works.
- [ ] `docker --version` works.
- [ ] `ffmpeg -version` works.
- [ ] The workspace root contains `primary_build_source/`.
- [ ] The editor trusts the workspace.
- [ ] `.vscode/settings.json` and `.vscode/extensions.json` exist.

## 🔧 Phase 2 - Bootstrap (First Run)

### 1. Create The Monorepo Folder Layout

Run in the workspace root:

```powershell
mkdir apps, packages, infra
mkdir apps\web, apps\api, apps\worker
mkdir packages\contracts, packages\db, packages\ui, packages\sim-core, packages\sim-renderer, packages\sim-physics, packages\sim-loto, packages\sim-telemetry
mkdir infra\docker
```

Success: those folders appear in the editor.

If it fails: if PowerShell says a folder already exists, continue. That is safe.

Git:

```powershell
git add apps packages infra
git commit -m "chore: scaffold monorepo folder structure"
```

### 2. Create The Root Workspace Files

Run in the workspace root:

```powershell
pnpm init
```

Then ask your AI copilot to create these files in one small batch:

- `package.json`
- `pnpm-workspace.yaml`
- `turbo.json`
- `tsconfig.base.json`
- `.gitignore`

Use this prompt:

```text
Using primary_build_source/docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md as the source of truth, create a minimal pnpm + turbo monorepo root for Myelin. Keep it small. Add workspaces for apps/* and packages/*. Add root scripts: dev, build, lint, typecheck, test, test:integration, test:sim, test:e2e, db:migrate, db:seed. Do not scaffold feature code yet. Preserve primary_build_source/ as a reference folder.
```

Success:

- `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.base.json`, and `.gitignore` exist.
- `pnpm-workspace.yaml` includes `apps/*` and `packages/*`.

If it fails:

- If your AI creates too much, revert only the extra files and ask for the same result in 3 files or fewer.
- If `pnpm init` fails, make sure `pnpm -v` works first.

Git:

```powershell
git add package.json pnpm-workspace.yaml turbo.json tsconfig.base.json .gitignore
git commit -m "chore: add monorepo root configuration"
```

### 3. Install Root Tooling

Run in the workspace root:

```powershell
pnpm add -Dw turbo typescript vite vitest @playwright/test eslint prettier @types/node tsx
pnpm exec playwright install
```

Success:

- `pnpm-lock.yaml` is created.
- Playwright installs browser binaries without error.

If it fails:

- If `pnpm` complains about a different lockfile, delete `package-lock.json` or `yarn.lock` if they exist and keep only `pnpm-lock.yaml`.
- If Playwright install fails, rerun it after confirming your internet connection.

Git:

```powershell
git add package.json pnpm-lock.yaml
git commit -m "chore: install root tooling and test dependencies"
```

### 4. Scaffold The Web App In Its Own Small Batch

Ask your AI copilot to create only the `apps/web` baseline first. Keep this change to 3 files or fewer in each copilot pass.

Use this prompt:

```text
Create the minimal Myelin web app in apps/web using React + Vite + TypeScript. Use primary_build_source/assets/styles/myelin-tokens.css as the styling reference. Add a single routed shell, not standalone module pages. Include a home route, a placeholder learner app route, and a visible note that the rebuild is in progress. Assume the app runs on http://localhost:3000 and talks to the API at VITE_API_BASE_URL.
```

Then install the web app dependencies:

```powershell
pnpm add --filter web react react-dom react-router-dom @tanstack/react-query three cannon-es gsap idb zod
pnpm add --filter web -D @types/react @types/react-dom @types/three
```

Run the web app:

```powershell
pnpm --filter web dev
```

Success:

- The web app starts.
- Opening `http://localhost:3000` shows a page instead of a blank screen or server error.

If it fails:

- If `No projects matched the filters` appears, the `apps/web/package.json` name is missing or incorrect.
- If port `3000` is already used, stop the other process or change the port in the Vite config and your `.env`.

Git:

```powershell
git add apps/web
git commit -m "feat: scaffold web app shell"
```

### 5. Scaffold The API In Its Own Small Batch

Ask your AI copilot to create only the API baseline next.

Use this prompt:

```text
Create the minimal Myelin API in apps/api using NestJS + Fastify + TypeScript. Add a /health route that returns JSON { "status": "ok" }. Keep the structure ready for tenant-aware modules later, but do not add auth or business logic yet. Assume the API runs on http://localhost:4000.
```

Install API dependencies:

```powershell
pnpm add --filter api @nestjs/common @nestjs/core @nestjs/platform-fastify @nestjs/config @fastify/cookie @fastify/multipart kysely pg pino bullmq ioredis zod
```

Run the API:

```powershell
pnpm --filter api dev
```

Open this URL in a browser:

```text
http://localhost:4000/health
```

Success: the browser returns `{"status":"ok"}` or equivalent pretty-printed JSON.

If it fails:

- If NestJS packages are missing, rerun the install command.
- If port `4000` is used, free that port or change the API port and update `apps/web/.env`.

Git:

```powershell
git add apps/api
git commit -m "feat: scaffold api health service"
```

### 6. Scaffold The Worker In Its Own Small Batch

Ask your AI copilot to create only the worker baseline next.

Use this prompt:

```text
Create a minimal Myelin worker in apps/worker using TypeScript. It should start cleanly, log that it is running, and be structured to accept BullMQ jobs later. Do not add real job handlers yet.
```

Install worker dependencies:

```powershell
pnpm add --filter worker bullmq ioredis pino pg kysely zod
```

Run the worker:

```powershell
pnpm --filter worker dev
```

Success: the terminal shows a startup log such as `worker started`.

If it fails:

- If the worker immediately crashes due to Redis connection logic, make the initial worker boot tolerant of missing Redis until Docker services are up.

Git:

```powershell
git add apps/worker
git commit -m "feat: scaffold worker process"
```

### 7. Add Shared Packages For Contracts, DB, UI, And Simulation

Create only package skeletons first. Do not fill them with feature code yet.

Use this prompt:

```text
Create minimal package.json and src entry files for packages/contracts, packages/db, packages/ui, packages/sim-core, packages/sim-renderer, packages/sim-physics, packages/sim-loto, and packages/sim-telemetry. Keep exports minimal and TypeScript-only. The goal is for the monorepo to build cleanly before feature logic is added.
```

Install the minimal package dependencies:

```powershell
pnpm add --filter contracts zod
pnpm add --filter db kysely pg uuid
pnpm add --filter sim-core zod
pnpm add --filter sim-telemetry zod uuid
pnpm add --filter sim-renderer three
pnpm add --filter sim-physics cannon-es
```

Success: every shared package has a `package.json` and `src` entry file.

If it fails: reduce the AI request so it creates only two or three packages at a time.

Git:

```powershell
git add packages
git commit -m "chore: add shared package skeletons"
```

### 8. Create Local Docker Services For Postgres And Redis

Ask your AI copilot to create `infra/docker/docker-compose.dev.yml` with:

- one Postgres service
- one Redis service
- named containers
- default local ports `5432` and `6379`
- a Postgres database named `myelin`

Use this prompt:

```text
Create infra/docker/docker-compose.dev.yml for Myelin local development. Add PostgreSQL and Redis only. Use simple local credentials suitable for development, not production. Name the database myelin.
```

Then run:

```powershell
docker compose -f infra/docker/docker-compose.dev.yml up -d
docker ps
```

Success:

- Docker lists running Postgres and Redis containers.
- No container exits immediately.

If it fails:

- If Docker is not running, start Docker Desktop first.
- If port `5432` or `6379` is busy, change the host port mapping and update your `.env` files.

Git:

```powershell
git add infra/docker/docker-compose.dev.yml
git commit -m "chore: add local postgres and redis services"
```

### 9. Create Your Environment Files

Create these files:

- `.env.example`
- `apps/web/.env`
- `apps/api/.env`
- `apps/worker/.env`

Use this environment table as your source of truth.

| Variable | Used By | Required For First Local Run | Source Basis | Safe Local Value |
|---|---|---:|---|---|
| `NODE_ENV` | API, worker | Yes | Confirmed by project docs example | `development` |
| `PORT` | API | Yes | Assumed | `4000` |
| `VITE_API_BASE_URL` | Web | Yes | Assumed | `http://localhost:4000` |
| `DATABASE_URL` | API, worker, DB package | Yes once DB code exists | Assumed from Postgres stack | `postgres://postgres:postgres@localhost:5432/myelin` |
| `REDIS_URL` | API, worker | Yes once queue/cache code exists | Assumed from Redis stack | `redis://localhost:6379` |
| `SESSION_COOKIE_SECRET` | API | No on day 1, yes before auth | Assumed from secure session requirements | a long random dev-only string |
| `JWT_PRIVATE_KEY_PATH` | API | No on day 1, yes before auth | Assumed from RS256 guidance | `./secrets/jwt-private.pem` |
| `JWT_PUBLIC_KEY_PATH` | API | No on day 1, yes before auth | Assumed from RS256 guidance | `./secrets/jwt-public.pem` |
| `EVIDENCE_BUCKET` | API, worker | No on day 1, yes before uploads | Confirmed by project docs example | `myelin-evidence-local` |
| `S3_ENDPOINT` | API, worker | Optional | Assumed from S3-compatible storage | empty until storage is added |
| `S3_REGION` | API, worker | Optional | Assumed | empty until storage is added |
| `S3_ACCESS_KEY_ID` | API, worker | Optional | Assumed | empty until storage is added |
| `S3_SECRET_ACCESS_KEY` | API, worker | Optional | Assumed | empty until storage is added |
| `WORKOS_CLIENT_ID` | API | Optional | Assumed from WorkOS recommendation | empty until SSO is added |
| `WORKOS_API_KEY` | API | Optional | Assumed from WorkOS recommendation | empty until SSO is added |
| `NVIDIA_API_KEY` | API or AI service | Optional | Assumed from NVIDIA-first AI direction | empty until AI demo is added |
| `SENTRY_DSN` | Web, API, worker | Optional | Assumed from observability recommendation | empty until monitoring is added |

Starter `.env.example`:

```dotenv
NODE_ENV=development
PORT=4000
VITE_API_BASE_URL=http://localhost:4000
DATABASE_URL=postgres://postgres:postgres@localhost:5432/myelin
REDIS_URL=redis://localhost:6379
SESSION_COOKIE_SECRET=change-me-before-sharing
JWT_PRIVATE_KEY_PATH=./secrets/jwt-private.pem
JWT_PUBLIC_KEY_PATH=./secrets/jwt-public.pem
EVIDENCE_BUCKET=myelin-evidence-local
S3_ENDPOINT=
S3_REGION=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
WORKOS_CLIENT_ID=
WORKOS_API_KEY=
NVIDIA_API_KEY=
SENTRY_DSN=
```

Starter `apps/web/.env`:

```dotenv
VITE_API_BASE_URL=http://localhost:4000
```

Starter `apps/api/.env`:

```dotenv
NODE_ENV=development
PORT=4000
DATABASE_URL=postgres://postgres:postgres@localhost:5432/myelin
REDIS_URL=redis://localhost:6379
SESSION_COOKIE_SECRET=change-me-before-sharing
EVIDENCE_BUCKET=myelin-evidence-local
```

Starter `apps/worker/.env`:

```dotenv
NODE_ENV=development
DATABASE_URL=postgres://postgres:postgres@localhost:5432/myelin
REDIS_URL=redis://localhost:6379
EVIDENCE_BUCKET=myelin-evidence-local
```

Success:

- All four files exist.
- No secrets are committed to the repo other than safe local placeholders.

If it fails:

- If your AI tries to invent extra env vars for providers not chosen yet, delete them and stay with the table above.

Git:

```powershell
git add .env.example apps/web/.env apps/api/.env apps/worker/.env
git commit -m "chore: add local environment configuration"
```

### 10. Add Database Migration And Seed Scripts

Ask your AI copilot to create only the minimal DB setup next.

Use this prompt:

```text
Using packages/db and apps/api, add a minimal Kysely setup, one initial migration, and root scripts pnpm db:migrate and pnpm db:seed. Keep the first schema tiny. The goal is only to prove the local database loop works.
```

Then run:

```powershell
pnpm db:migrate
pnpm db:seed
```

Success:

- The commands exit without error.
- The database contains the first schema and any seed rows you intentionally created.

If it fails:

- Confirm Postgres is running with `docker ps`.
- Confirm `DATABASE_URL` matches the Postgres container port and database name.

Git:

```powershell
git add packages/db apps/api package.json
git commit -m "feat: add initial database migration and seed flow"
```

### 11. Run The Full Local Stack For The First Time

Run in the workspace root:

```powershell
pnpm dev
```

Then open:

- `http://localhost:3000`
- `http://localhost:4000/health`

Run the first quality pass:

```powershell
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

Success:

- `pnpm dev` starts the web app, API, and worker.
- The web app loads in the browser.
- The API health route responds.
- Lint, typecheck, tests, and build finish without blocking errors.

If it fails:

- If `pnpm dev` cannot find a script, check the root `package.json`.
- If the web app cannot reach the API, confirm `VITE_API_BASE_URL`.
- If the build fails, fix TypeScript path and export errors before continuing.

Git:

```powershell
git add .
git commit -m "feat: complete first local monorepo run"
```

### Common Bootstrap Problems And Fixes

| Problem | What It Usually Means | Fix |
|---|---|---|
| `pnpm: command not found` | Corepack was not enabled | Run `corepack enable` and `corepack prepare pnpm@latest --activate` |
| `No projects matched the filters` | Workspace package name or filter target is wrong | Open the package's `package.json` and confirm its `name` field |
| `EADDRINUSE` on port `3000`, `4000`, `5432`, or `6379` | Another process is already using the port | Stop that process or remap the port and update `.env` |
| Playwright install fails | Browser binaries were not downloaded | Run `pnpm exec playwright install` again |
| API cannot connect to Postgres | Docker is down or `DATABASE_URL` is wrong | Run `docker ps` and confirm the DB URL |
| Web app shows blank page | Vite, React entry, or route setup is broken | Check the browser console and confirm `src/main.tsx` exists |
| Worker crashes at boot | It expects Redis or DB before those services are available | Make the initial worker boot tolerant and retry-friendly |

### First Run Checklist

- [ ] `pnpm dev` starts all local services.
- [ ] `http://localhost:3000` loads the web shell.
- [ ] `http://localhost:4000/health` returns healthy JSON.
- [ ] Docker shows Postgres and Redis containers running.
- [ ] `pnpm lint` passes.
- [ ] `pnpm typecheck` passes.
- [ ] `pnpm test` passes.
- [ ] `pnpm build` passes.
- [ ] Git history contains small setup commits, not one huge bootstrap commit.

## 🔄 Phase 3 - Vibe-Coding Loop (Safe Iteration)

The safest loop for this project is:

1. Plan one change.
2. Touch 1 or 2 files.
3. Run the smallest relevant test.
4. Inspect the diff.
5. Commit only after the change works.

Do not ask your AI copilot to edit 10 files at once. Myelin has too many hard constraints for that to stay safe.

### 3.1 Decision Framework (What To Build Next)

Use this priority order:

1. Foundation first:
   - monorepo health
   - shared contracts
   - DB and tenant isolation
   - routed app shell
2. Core user path next:
   - learner dashboard shell
   - module routing
   - simulation host page
3. Simulation correctness before polish:
   - state machine
   - thresholds
   - telemetry
   - wrong-order blocking
4. AI text mode before live video:
   - context binding
   - guardrails
   - transcript logging
5. Evidence workflows after the core path is stable:
   - uploads
   - peer review
   - workplace logbook
   - result statements

When deciding whether something is the next task, ask:

- Does it unblock the main learner journey?
- Does it protect a locked rule from the docs?
- Can I finish it in one small commit?

### 3.2 File Location Guide (Where Things Live)

Use this table when deciding where to work.

| Location | Use It For |
|---|---|
| `primary_build_source/docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md` | Product scope, roles, journeys, and non-functional goals |
| `primary_build_source/docs/v1_decisions.md` | Locked v1 decisions that your code must not violate |
| `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md` | Simulation performance, physics, thresholds, and telemetry rules |
| `primary_build_source/docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md` | Monorepo shape, app/package layout, dependency choices |
| `primary_build_source/assets/styles/myelin-tokens.css` | UI token names and theme behavior |
| `apps/web/src/routes` | Pages and route-level screens |
| `apps/web/src/features` | UI features like pathway, submissions, profile, AI panel |
| `apps/web/src/simulator-host` | The React surface that mounts the simulator |
| `apps/api/src/modules` | Domain APIs such as auth, learning, telemetry, results, AI |
| `packages/contracts/src` | Shared schemas and types |
| `packages/db/src` | Schema, migrations, RLS helpers, query modules |
| `packages/sim-core/src` | State machine, scoring, session flow |
| `packages/sim-renderer/src` | Three.js renderer, environment, post-processing |
| `packages/sim-physics/src` | `cannon-es` world setup and constraints |
| `packages/sim-loto/src` | LOTO-specific objects, thresholds, and interactions |
| `packages/sim-telemetry/src` | Immutable simulation events and replay helpers |

### 3.3 Five Ready-To-Copy Prompts

Use these in Cursor or VS Code chat. Always paste the file paths you want the AI to work in.

1. Architecture explanation

```text
You are my coding copilot for the Myelin rebuild. Using primary_build_source/docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md, primary_build_source/docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md, and the current apps/ and packages/ folders, explain the architecture to me like I am non-technical. Keep it simple and map each major responsibility to a real folder.
```

2. Small-batch implementation plan

```text
I want to add [FEATURE]. Based on this Myelin codebase, propose a step-by-step plan that touches no more than 3 files at once. Use the existing folders only. For each step, tell me which test to run before moving on.
```

3. Minimal-diff bug fix

```text
Given this error log: [PASTE ERROR]. Find the likely root cause in apps/web, apps/api, or packages/* and propose the smallest safe change. Do not refactor unrelated code. Tell me exactly what to test after the fix.
```

4. Convention check

```text
Review this change against Myelin's source rules in primary_build_source/docs/v1_decisions.md and primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md. Call out anything that breaks routed-app structure, tenant isolation, LOTO thresholds, demo/live separation, or the post-simulator route /app/submissions/new.
```

5. Test generation

```text
Generate tests for this code using the existing Myelin test stack. Use Vitest for unit logic, integration tests for API + DB behavior, and Playwright only for full user journeys. Keep test names behavior-focused and match the project's tenant-isolation and LOTO-step rules.
```

### 3.4 Safety Checklists

#### Before Code Changes

- [ ] I can name the exact behavior I am changing.
- [ ] I know which 1 to 3 files I expect to touch first.
- [ ] I checked the matching source doc in `primary_build_source/docs/`.
- [ ] I know which command I will run to test this change.
- [ ] I know the commit message I plan to use if it works.

#### After AI Suggestions

- [ ] The AI changed only the files I expected.
- [ ] The AI did not reintroduce standalone module pages.
- [ ] The AI did not invent new role names or tenant rules.
- [ ] The AI did not change LOTO thresholds or simulator physics constants without source backing.
- [ ] The AI did not route simulator completion anywhere except `/app/submissions/new`.
- [ ] The AI did not add secrets to tracked files.

#### Before Git Commit

- [ ] `git diff --stat` shows a small, understandable change.
- [ ] Relevant tests passed.
- [ ] The app still starts if the change affects startup.
- [ ] New env vars, scripts, or folders are documented.
- [ ] The commit message describes behavior, not just files.

### The Daily Loop: Plan -> Implement -> Inspect -> Test -> Commit

1. Check your branch and working tree.

```powershell
git status
git branch --show-current
```

2. Ask the AI for a small plan tied to real files.

3. Apply the change in a tiny batch.

4. Inspect only your change.

```powershell
git diff --stat
git diff
```

5. Run the smallest relevant test.

Examples:

```powershell
pnpm lint
pnpm typecheck
pnpm test
pnpm test:integration
pnpm test:sim
pnpm test:e2e
```

6. Commit when green.

```powershell
git add .
git commit -m "feat: add learner dashboard route shell"
```

7. Push only after a clean local run.

```powershell
git push origin <your-branch-name>
```

## 🧪 Phase 4 - Testing Matrix

The Myelin source package explicitly expects unit, integration, E2E, and simulator-specific testing, plus release-gate validation. Use this matrix as the default rule.

| Test Type | Command | When | Green = |
|---|---|---|---|
| Lint | `pnpm lint` | Before every commit | `0` errors |
| Typecheck | `pnpm typecheck` | Before every commit | `0` TypeScript errors |
| Unit | `pnpm test` | After every logic change | All Vitest unit specs pass |
| Integration | `pnpm test:integration` | Before merging to `main` | API + DB + queue behavior passes with real test services |
| Simulator-specific | `pnpm test:sim` | Before staging and before simulator releases | LOTO thresholds, sequence gating, telemetry, and perf rules pass |
| E2E | `pnpm test:e2e` | Before staging release or demo handoff | Learner, lecturer, and admin journeys pass |
| Build | `pnpm build` | Before push and before deploy | All apps/packages build cleanly and expected `dist` output exists |

### What A Good Test Run Looks Like

- Lint: no red errors.
- Typecheck: no missing types, imports, or path alias failures.
- Unit tests: clear `passed` summary and no skipped critical tests.
- Integration tests: the database, queue, and API all work together.
- Simulator tests: exact threshold values behave correctly.
- E2E tests: the browser opens, clicks through the flow, and exits cleanly.
- Build: the repo produces production artifacts with no compile-time failures.

### Common Failure Patterns And What To Do Next

| Failure | Likely Cause | Next Step |
|---|---|---|
| Lint fails | Style drift or unused code | Fix the file instead of disabling lint rules |
| Typecheck fails | Broken imports, env typing, schema mismatch | Fix the type error before testing behavior |
| Unit test fails | Logic regression | Read the failing test first, then compare behavior to the source docs |
| Integration test fails | Postgres or Redis is down, or schema drift exists | Re-run Docker services, then rerun migrations |
| Simulator test fails | Threshold or state-machine rule changed | Check `primary_build_source/docs/MYELIN_SIMULATION_BUILD_SINGLE_SOURCE_SPEC.md` |
| E2E test fails | Route mismatch, missing seed data, or startup issue | Reproduce locally in the browser before changing code |
| Build fails | Broken exports or missing package references | Fix workspace wiring before adding features |

### Quality Checklist

- [ ] `pnpm lint` passes.
- [ ] `pnpm typecheck` passes.
- [ ] `pnpm test` passes.
- [ ] `pnpm test:integration` passes before merge.
- [ ] `pnpm test:sim` passes before simulator demos or releases.
- [ ] `pnpm test:e2e` passes before staging release.
- [ ] `pnpm build` passes.
- [ ] No secrets were added to tracked files.
- [ ] Tenant isolation rules were not weakened.
- [ ] Demo mode still stays isolated from live data.

## 🚀 Phase 5 - Build & Deploy

### 1. Create A Production Build

Run in the workspace root:

```powershell
pnpm build
```

Success:

- `apps/web` produces a web build, usually in `dist/`.
- `apps/api` produces compiled server output, usually in `dist/`.
- `apps/worker` produces compiled worker output, usually in `dist/`.

If it fails:

- Fix TypeScript, import, and workspace export errors before retrying.
- Do not deploy a partial build.

Git:

```powershell
git add .
git commit -m "chore: verify production build output"
```

### 2. Know The Dev / Stage / Prod Differences

| Environment | Purpose | Expected Differences |
|---|---|---|
| `dev` | Local daily coding | Local `.env` files, Docker Postgres/Redis, feature work in progress |
| `stage` | Pre-release validation | Production-like config, test tenants, release-gate checks, no live learner data |
| `prod` | Live institutional use | Managed secrets, managed Postgres/Redis/storage, monitoring, rollback path, strict sign-off |

Rules:

1. Never point demo mode at live data.
2. Never commit production secrets to the repo.
3. Run the release gates before promoting a build.

### 3. Deployment Assumption For This Project

The source package does not provide Dockerfiles or platform-specific deployment manifests. The safest default assumption is:

1. Build `apps/web`, `apps/api`, and `apps/worker` separately.
2. Deploy them through a container-based pipeline.
3. Use managed Postgres and Redis in stage and prod.
4. Use S3-compatible storage for uploads and evidence.
5. Keep secrets in a secrets manager, not in committed env files.
6. Keep a previous-release redeploy path ready.

If you later choose Vercel or Netlify for the web app, remember:

- only the web app naturally fits there
- the API and worker still need backend hosting
- Postgres, Redis, and storage still need their own infrastructure

### 4. Release Gates You Must Respect

The project docs define blocking release gates. Before any production deployment, you should have evidence for:

- Security:
  - SAST
  - DAST
  - dependency scan
  - secret scan
- Performance:
  - simulator load time
  - frame rate
  - API response time
- Compliance:
  - POPIA consent flow
  - retention job behavior
  - audit log coverage
  - cross-tenant isolation
- Curriculum integrity:
  - all 7 LOTO steps
  - curriculum outcome mapping
  - demo mode isolation
- Product acceptance:
  - learner journey
  - lecturer journey
  - agent safety walkthrough

### 5. Pre-Deployment Checklist

- [ ] `pnpm build` passes.
- [ ] Lint, typecheck, unit, integration, simulator, and E2E tests all pass at the right stage.
- [ ] Demo data is isolated from live data.
- [ ] Tenant isolation has been tested.
- [ ] Consent flow and audit logging are verified.
- [ ] The simulator still enforces all 7 LOTO steps correctly.
- [ ] The post-simulator route still goes to `/app/submissions/new`.
- [ ] Secrets are stored outside the repo.
- [ ] A rollback path exists.
- [ ] The release notes explain what changed in user behavior.

## 📚 Appendix - Copilot Prompts

These are copy-paste prompts tailored to this project. Replace bracketed text before sending.

### 1. Explain The Project To Me Like I'm Non-Technical

```text
Using primary_build_source/docs/MYELIN_PROFESSIONAL_REBUILD_PRD.md, primary_build_source/docs/v1_decisions.md, and the current apps/ and packages/ folders, explain Myelin to me in plain language. Tell me what the web app, API, worker, simulator packages, and database package each do.
```

### 2. Walk Me Through The Code Path When The User Does X

```text
Walk me through the code path when the user [DESCRIBE ACTION], starting from apps/web and ending in apps/api or packages/*. Name the files in order, explain what each one does, and point out where tenant isolation, demo mode, or simulator rules apply.
```

### 3. Propose A Safe Refactor With Step-By-Step Commits

```text
Propose a safe refactor for [FILE OR MODULE] in the Myelin monorepo. Keep behavior identical. Break the work into commit-sized steps that touch no more than 3 files at once. Include the test command after each step.
```

### 4. Audit A Module For Bugs And Edge Cases

```text
Audit [FILE OR MODULE] for potential bugs, regressions, and missing edge cases. Prioritize tenant isolation, demo/live separation, LOTO step-order rules, and the required route /app/submissions/new after simulator completion. Findings first, summary second.
```

### 5. Generate Tests Using The Existing Framework

```text
Generate tests for [FUNCTION OR MODULE] using Myelin's expected test stack: Vitest for unit logic, integration tests for API + DB behavior, and Playwright only if the feature is a full user journey. Use behavior-focused test names and include exact commands to run them.
```

### 6. Fix An Error With A Minimal Diff

```text
Fix this error in the Myelin monorepo with the smallest safe diff: [PASTE ERROR]. Search only the most likely files first. Explain the root cause, show the exact files to change, and tell me the shortest test command to prove the fix.
```

### 7. Check Whether A Change Matches Project Conventions

```text
Review this code against Myelin's source rules in primary_build_source/docs/MYELIN_V1_RECOMMENDED_TECH_STACK.md, primary_build_source/docs/v1_decisions.md, and primary_build_source/assets/styles/myelin-tokens.css. Call out any mismatch in folder placement, route design, theme token usage, tenant isolation, or simulator constraints.
```

### 8. Update The Project Docs After A Change

```text
Update the relevant Myelin docs to reflect the current implementation. Compare the code in apps/, packages/, and infra/ against primary_build_source/docs/. Write a concise doc update that explains what is now implemented, what is still assumed, and what commands a new developer should run first.
```
