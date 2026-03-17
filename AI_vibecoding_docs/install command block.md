# Windows day-1 bootstrap commands for this workspace
# Run from: C:\Users\ssnguna\Local Sites\myelin-platform

# 1) Verify machine prerequisites
git --version
node -v
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v
docker --version
docker info

# 2) Create target rebuild folders (safe if they already exist)
mkdir apps, packages, infra -ErrorAction SilentlyContinue
mkdir apps\web, apps\api, apps\worker -ErrorAction SilentlyContinue
mkdir packages\contracts, packages\db, packages\ui, packages\sim-core, packages\sim-renderer, packages\sim-physics, packages\sim-loto, packages\sim-telemetry, packages\ai-orchestration -ErrorAction SilentlyContinue
mkdir infra\docker -ErrorAction SilentlyContinue

# 3) Initialize root workspace (if package.json does not exist yet)
if (!(Test-Path package.json)) { pnpm init }

# 4) Install day-1 root tooling
pnpm add -Dw turbo typescript vite vitest @playwright/test eslint prettier @types/node tsx
pnpm exec playwright install

# 5) Add core backend/runtime libs (after app package.json files exist)
# pnpm add --filter api @nestjs/common @nestjs/core @nestjs/platform-fastify @nestjs/config kysely pg pino bullmq ioredis zod
# pnpm add --filter worker bullmq ioredis pino kysely pg zod
# pnpm add --filter web react react-dom react-router-dom @tanstack/react-query zod

# 6) Start local infra once compose file exists
# docker compose -f infra/docker/docker-compose.dev.yml up -d
# docker ps

# 7) First quality loop once scripts exist
# pnpm dev
# pnpm lint
# pnpm typecheck
# pnpm test
# pnpm build