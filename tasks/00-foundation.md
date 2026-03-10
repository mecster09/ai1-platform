# 00 Foundation

## Goal

Set up the monorepo, shared TypeScript tooling, local infrastructure bootstrap, and runtime directories.

## Target Stack

- Workspace: `pnpm`
- Language: `TypeScript`
- Front end: `Next.js`
- Backend and services: `Node.js`
- Workflow engine: `Temporal`
- Local retrieval store: `ChromaDB`

## Tasks

### T-001 Monorepo Workspace

- Outcome: A working root workspace with `apps/`, `agents/`, `packages/`, `infra/`, `scripts/`, and `docs/`.
- Dependencies: none

- [ ] Initialize `pnpm` workspace files at the repo root
- [ ] Add root `package.json`
- [ ] Add `pnpm-workspace.yaml`
- [ ] Add root scripts for `dev`, `build`, `lint`, and `test`
- [ ] Verify workspace package discovery works

### T-002 Repo Structure

- Outcome: Source tree matches the intended monorepo shape.
- Dependencies: `T-001`

- [ ] Create `apps/`, `agents/`, `packages/`, `infra/`, `scripts/`, and `docs/`
- [ ] Create initial app folders for `platform-web`, `platform-api`, `workflow-worker`, `tasks-agent-worker`, and `architect-agent-worker`
- [ ] Create initial shared package folders
- [ ] Confirm the structure matches [repo-structure.md](c:/Code/AI1-Platform/docs/repo-structure.md)

### T-003 Shared TypeScript and Tooling

- Outcome: Consistent build and lint behavior across all apps and packages.
- Dependencies: `T-001`

- [ ] Add shared `tsconfig.base.json`
- [ ] Add ESLint configuration for Node.js and Next.js packages
- [ ] Add Prettier configuration
- [ ] Add path alias conventions for shared packages
- [ ] Verify lint and typecheck run from the root

### T-004 Local Infrastructure Bootstrap

- Outcome: Developers can boot the local platform infrastructure with one command.
- Dependencies: `T-001`

- [ ] Add local infra definitions for `Temporal`
- [ ] Add local infra definitions for `ChromaDB`
- [ ] Add startup scripts for local development
- [ ] Add environment variable templates
- [ ] Add provider configuration variables for `AI_PROVIDER`, `AI_MODEL`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and `OPENAI_BASE_URL`
- [ ] Document where `.env` files should live for platform services and agent workers
- [ ] Verify infrastructure can be started locally

### T-005 Runtime State and Git Ignore

- Outcome: Runtime state is isolated from source-controlled code.
- Dependencies: `T-002`

- [ ] Create `artifacts/`, `workspaces/`, and `data/` directories
- [ ] Update `.gitignore` for runtime state, logs, coverage, and generated outputs
- [ ] Add placeholder `.gitkeep` files only where useful
- [ ] Verify runtime state is not tracked by Git
