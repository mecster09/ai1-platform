# AI1 Platform Repo Structure

## Purpose

This document recommends a monorepo layout for the AI1 Platform based on the target decomposition in [plan.md](c:/Code/AI1-Platform/docs/plan.md).

The structure is designed to support:

- A local `Next.js` platform UI
- A `Node.js` platform API
- `Temporal` workflow workers
- Specialist agent workers
- Shared typed contracts and schemas
- Local-first infrastructure with `SQLite`, `ChromaDB`, filesystem artifacts, and isolated workspaces

## Design Principles

- Keep platform services and agent workers as separate deployable apps
- Centralize shared contracts, schemas, and workflow types in packages
- Keep infrastructure definitions local and explicit
- Separate generated runtime state from source-controlled code
- Make the MVP easy to run locally before optimizing for scale

## Recommended Top-Level Layout

```text
/apps
/packages
/agents
/workflows
/infra
/scripts
/docs
/artifacts
/workspaces
/data
package.json
pnpm-workspace.yaml
tsconfig.base.json
```

## Recommended Monorepo Tree

```text
/
|-- apps/
|   |-- platform-web/
|   |   |-- app/
|   |   |-- components/
|   |   |-- lib/
|   |   |   |-- api-client/
|   |   |   |-- actions/
|   |   |   `-- telemetry/
|   |   |-- public/
|   |   |-- styles/
|   |   |-- tests/
|   |   |-- instrumentation.ts
|   |   |-- next.config.ts
|   |   `-- package.json
|   |
|   |-- platform-api/
|   |   |-- src/
|   |   |   |-- modules/
|   |   |   |   |-- stories/
|   |   |   |   |-- artifacts/
|   |   |   |   |-- runs/
|   |   |   |   |-- reviews/
|   |   |   |   `-- traceability/
|   |   |   |-- commands/
|   |   |   |-- queries/
|   |   |   |-- streaming/
|   |   |   |-- artifacts/
|   |   |   |-- routes/
|   |   |   |-- services/
|   |   |   |-- repositories/
|   |   |   |-- db/
|   |   |   |-- events/
|   |   |   `-- main.ts
|   |   `-- package.json
|   |
|   |-- workflow-worker/
|   |   |-- src/
|   |   |   |-- workflows/
|   |   |   |-- activities/
|   |   |   |-- signals/
|   |   |   `-- worker.ts
|   |   `-- package.json
|   |
|   |-- context-service/
|   |   |-- src/
|   |   |   |-- indexing/
|   |   |   |-- retrieval/
|   |   |   |-- embeddings/
|   |   |   `-- server.ts
|   |   `-- package.json
|   |
|   `-- workspace-service/
|       |-- src/
|       |   |-- git/
|       |   |-- execution/
|       |   |-- isolation/
|       |   |-- retention/
|       |   `-- server.ts
|       `-- package.json
|
|-- agents/
|   |-- tasks-agent-worker/
|   |   |-- src/
|   |   |   |-- prompts/
|   |   |   |-- tools/
|   |   |   |-- mappers/
|   |   |   `-- worker.ts
|   |   `-- package.json
|   |
|   |-- architect-agent-worker/
|   |   |-- src/
|   |   |   |-- prompts/
|   |   |   |-- tools/
|   |   |   |-- generators/
|   |   |   `-- worker.ts
|   |   `-- package.json
|   |
|   |-- frontend-agent-worker/
|   |   |-- src/
|   |   |   |-- prompts/
|   |   |   |-- tools/
|   |   |   `-- worker.ts
|   |   `-- package.json
|   |
|   |-- backend-agent-worker/
|   |   |-- src/
|   |   |   |-- prompts/
|   |   |   |-- tools/
|   |   |   `-- worker.ts
|   |   `-- package.json
|   |
|   `-- test-agent-worker/
|       |-- src/
|       |   |-- prompts/
|       |   |-- tools/
|       |   |-- playwright/
|       |   `-- worker.ts
|       `-- package.json
|
|-- packages/
|   |-- contracts/
|   |   |-- src/
|   |   |   |-- story.ts
|   |   |   |-- task.ts
|   |   |   |-- architecture-blueprint.ts
|   |   |   |-- api-contract.ts
|   |   |   |-- data-model-change.ts
|   |   |   |-- code-change-set.ts
|   |   |   |-- test-scenario.ts
|   |   |   |-- agent-run-result.ts
|   |   |   |-- review-decision.ts
|   |   |   |-- run-event.ts
|   |   |   `-- artifact-reference.ts
|   |   `-- package.json
|   |
|   |-- schemas/
|   |   |-- src/
|   |   |   |-- story.schema.json
|   |   |   |-- acceptance-criterion.schema.json
|   |   |   |-- task.schema.json
|   |   |   |-- architecture-blueprint.schema.json
|   |   |   |-- api-contract.schema.json
|   |   |   |-- data-model-change.schema.json
|   |   |   |-- code-change-set.schema.json
|   |   |   |-- test-scenario.schema.json
|   |   |   |-- agent-run-result.schema.json
|   |   |   `-- review-decision.schema.json
|   |   `-- package.json
|   |
|   |-- database/
|   |   |-- src/
|   |   |   |-- sqlite/
|   |   |   |-- migrations/
|   |   |   |-- repositories/
|   |   |   `-- client.ts
|   |   `-- package.json
|   |
|   |-- context-model/
|   |   |-- src/
|   |   |   |-- chroma/
|   |   |   |-- chunking/
|   |   |   |-- indexing/
|   |   |   `-- queries/
|   |   `-- package.json
|   |
|   |-- workflow-types/
|   |   |-- src/
|   |   |   |-- commands.ts
|   |   |   |-- activities.ts
|   |   |   |-- signals.ts
|   |   |   `-- task-queues.ts
|   |   `-- package.json
|   |
|   |-- platform-api-client/
|   |   |-- src/
|   |   |   |-- commands/
|   |   |   |-- queries/
|   |   |   |-- streaming/
|   |   |   `-- artifacts/
|   |   `-- package.json
|   |
|   |-- agent-runtime/
|   |   |-- src/
|   |   |   |-- runner/
|   |   |   |-- tool-contracts/
|   |   |   |-- output-contracts/
|   |   |   `-- policies/
|   |   `-- package.json
|   |
|   |-- observability/
|   |   |-- src/
|   |   |   |-- logging/
|   |   |   |-- tracing/
|   |   |   `-- metrics/
|   |   `-- package.json
|   |
|   `-- config/
|       |-- src/
|       |   |-- env.ts
|       |   |-- paths.ts
|       |   `-- feature-flags.ts
|       `-- package.json
|
|-- workflows/
|   |-- create-story/
|   |-- decompose-story/
|   |-- create-architecture/
|   |-- implement-frontend/
|   |-- implement-backend/
|   |-- generate-e2e-tests/
|   |-- validate-delivery/
|   `-- review-and-merge/
|
|-- infra/
|   |-- docker/
|   |   |-- temporal/
|   |   |-- chromadb/
|   |   `-- dev/
|   |-- compose/
|   |   `-- docker-compose.local.yml
|   |-- temporal/
|   |   |-- namespaces/
|   |   `-- task-queues/
|   `-- db/
|       |-- sqlite/
|       `-- seeds/
|
|-- scripts/
|   |-- dev/
|   |-- bootstrap/
|   |-- migrate/
|   |-- index-context/
|   `-- validate/
|
|-- docs/
|   |-- plan.md
|   |-- architecture.md
|   |-- schemas.md
|   |-- repo-structure.md
|   `-- architecture/
|
|-- artifacts/
|   |-- stories/
|   |-- runs/
|   |-- reviews/
|   `-- evidence/
|
|-- workspaces/
|   `-- {storyId}/
|       `-- {runId}/
|           `-- {agentType}/
|
`-- data/
    |-- sqlite/
    |   `-- ai1-platform.db
    `-- chromadb/
```

## Directory Responsibilities

### `apps/`

Holds the long-lived platform services identified in the plan:

- `platform-web`
- `platform-api`
- `workflow-worker`
- `context-service`
- `workspace-service`

These are the primary runtime processes of the platform and should be deployable independently.

### `agents/`

Holds each specialist worker as its own app:

- `tasks-agent-worker`
- `architect-agent-worker`
- registered delivery-agent workers
- current built-in delivery workers:
- `frontend-agent-worker`
- `backend-agent-worker`
- `test-agent-worker`

This keeps agent-specific prompts, tool wiring, and policies isolated while still sharing contracts and runtime libraries from `packages/`.

### `packages/`

Holds shared code that multiple services depend on:

- Domain contracts
- JSON schemas
- SQLite access and migrations
- ChromaDB retrieval support
- Workflow command and signal types
- Agent runtime abstractions
- Logging and configuration

This is the main boundary that prevents copy-paste logic across platform services and agents.

### `workflows/`

Holds the business workflow definitions called out in the plan:

- `CreateStoryWorkflow`
- `DecomposeStoryWorkflow`
- `CreateArchitectureWorkflow`
- selected delivery workflows for the story
- current built-in delivery workflows:
- `ImplementFrontendWorkflow`
- `ImplementBackendWorkflow`
- `GenerateE2ETestsWorkflow`
- `ValidateDeliveryWorkflow`
- `ReviewAndMergeWorkflow`

This folder is useful when workflow definitions are substantial enough to deserve their own structure outside the raw `workflow-worker` bootstrap.

### `infra/`

Holds local infrastructure configuration for:

- `Temporal`
- `ChromaDB`
- Docker compose
- SQLite initialization and seed data

Keep this folder operational and environment-focused, not application-focused.

### `scripts/`

Holds developer and CI helper scripts such as:

- Local startup
- DB migration
- Context indexing
- Validation and smoke checks
- Artifact cleanup

### `artifacts/`

Holds generated runtime outputs that should not live inside app folders:

- Uploaded attachments
- Generated architecture packs
- Patch files
- Validation logs
- Test evidence
- Review bundles

This mirrors the artifact model described in the architecture docs.

### `workspaces/`

Holds isolated repo work areas created by `workspace-service`:

```text
/workspaces/{storyId}/{runId}/{agentType}/
```

This must remain runtime state, not source-controlled code.

### `data/`

Holds local persistent service data:

- `SQLite` database file
- `ChromaDB` local collections and indexes

This makes the local-first runtime explicit and easy to back up or reset.

## Package Guidance

Recommended internal package split:

- `@ai1/contracts`
- `@ai1/schemas`
- `@ai1/database`
- `@ai1/context-model`
- `@ai1/workflow-types`
- `@ai1/agent-runtime`
- `@ai1/observability`
- `@ai1/config`

Keep these packages small and deliberate. Do not create many micro-packages early.

## App Guidance

Recommended app ownership:

- `platform-web`: intake UI, review UI, traceability views, logs, artifacts, status timelines
- `platform-api`: command, query, streaming, and artifact boundary; persistence; approvals; workflow initiation
- `workflow-worker`: Temporal worker bootstrap and workflow registration
- `context-service`: indexing, retrieval, source tracking, ChromaDB query orchestration
- `workspace-service`: git operations, worktree management, command execution, log capture

Boundary guidance:

- `platform-web` should use shared typed API clients or Server Actions that preserve the public API semantics.
- `platform-api` should keep command, query, streaming, and artifact concerns explicit in code layout.
- Streaming support should live under a dedicated module so live run events and log tails do not get mixed into synchronous request handlers.

Recommended agent ownership:

- `tasks-agent-worker`: task decomposition only
- `architect-agent-worker`: blueprint and machine-readable contracts only
- delivery agents are selected per story from the task breakdown
- `frontend-agent-worker`: UI changes only
- `backend-agent-worker`: API, service, and data-layer changes only
- `test-agent-worker`: `Playwright` generation and test mapping only

## Source Control Guidance

Commit to source control:

- `apps/`
- `agents/`
- `packages/`
- `workflows/`
- `infra/`
- `scripts/`
- `docs/`

Do not commit runtime state:

- `artifacts/`
- `workspaces/`
- `data/`
- coverage outputs
- logs
- temporary generated bundles

## Recommended `.gitignore` Areas

At minimum, ignore:

```text
artifacts/
workspaces/
data/
.turbo/
dist/
.next/
coverage/
playwright-report/
test-results/
*.log
```

## MVP Cut

For MVP 1, the repo can start smaller:

```text
/apps/platform-web
/apps/platform-api
/apps/workflow-worker
/agents/tasks-agent-worker
/agents/architect-agent-worker
/packages/contracts
/packages/schemas
/packages/database
/packages/workflow-types
/packages/config
/docs
/infra
/scripts
```

Add configurable delivery-agent selection, the remaining delivery agents, context-service, and workspace-service structure as MVP 2 and MVP 3 land.

## Why This Layout Fits The Plan

- It matches the service inventory in the platform decomposition.
- It preserves strict separation between platform control-plane services and specialist agents.
- It gives contracts and schemas a first-class place in the repo, which the plan treats as critical.
- It supports local runtime state for `SQLite`, `ChromaDB`, artifacts, and workspaces without mixing that state into source code.
- It keeps the MVP simple enough to run locally while still leaving room for future scaling.
