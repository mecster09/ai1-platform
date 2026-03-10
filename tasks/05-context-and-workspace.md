# 05 Context And Workspace

## Goal

Build the retrieval and execution services that make agent runs scoped, reproducible, and isolated.

## Target Stack

- Services: `Node.js` + `TypeScript`
- Metadata store: `SQLite`
- Vector retrieval: `ChromaDB`
- Repo execution: Git + local filesystem workspaces
- Runtime contract source: [architecture.md](c:/Code/AI1-Platform/docs/architecture.md)

## Tasks

### T-032 Shared Context Model

- Outcome: Shared context indexing and retrieval logic exists.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`, `tasks/01-contracts-and-persistence.md#t-010-initial-sqlite-schema`

- [ ] Create `@ai1/context-model`
- [ ] Add chunking utilities
- [ ] Add indexing metadata models
- [ ] Add ChromaDB collection helpers
- [ ] Add query and ranking helpers

### T-033 Context Service

- Outcome: Agents can retrieve scoped context instead of full-repo dumps.
- Dependencies: `T-032`

- [ ] Scaffold `apps/context-service`
- [ ] Add indexing pipeline for repo docs
- [ ] Add indexing pipeline for architecture docs
- [ ] Add indexing pipeline for coding standards
- [ ] Add indexing pipeline for API contracts and prior story artifacts
- [ ] Persist context metadata in SQLite

### T-034 Retrieval Interfaces

- Outcome: Agent workflows can request relevant context deterministically.
- Dependencies: `T-033`

- [ ] Implement story-scoped retrieval
- [ ] Implement module-scoped retrieval
- [ ] Implement contract-scoped retrieval
- [ ] Implement historical artifact retrieval
- [ ] Record retrieval audit trails

### T-035 Workspace Service

- Outcome: Agent execution is isolated and auditable.
- Dependencies: `tasks/00-foundation.md#t-004-local-infrastructure-bootstrap`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`

- [ ] Scaffold `apps/workspace-service`
- [ ] Add repo open or clone support
- [ ] Add branch creation support
- [ ] Add run workspace creation
- [ ] Add cleanup and retention hooks

### T-036 Workspace Isolation

- Outcome: Concurrent agent execution does not corrupt repo state.
- Dependencies: `T-035`

- [ ] Enforce one branch per agent run
- [ ] Enforce workspace paths under `/workspaces/{storyId}/{runId}/{agentType}`
- [ ] Prevent concurrent write collisions
- [ ] Capture workspace metadata in SQLite

### T-037 Safe Command Execution

- Outcome: Build, test, and patch application are usable by agents and validation.
- Dependencies: `T-035`

- [ ] Add targeted command execution
- [ ] Capture stdout and stderr
- [ ] Capture exit codes and timing
- [ ] Enforce timeouts
- [ ] Store command logs as artifacts
