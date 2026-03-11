# 01 Contracts And Persistence

## Goal

Create the shared contract layer, schema validation, SQLite persistence, and repository interfaces.

## Target Stack

- Language: `TypeScript`
- Backend libraries: `Node.js`
- Metadata store: `SQLite`
- Contract validation: JSON Schema
- Shared packages: `@ai1/contracts`, `@ai1/schemas`, `@ai1/database`

## Tasks

### T-006 TypeScript Domain Contracts

- Outcome: All services and agents consume one typed contract layer.
- Dependencies: `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Create `@ai1/contracts`
- [ ] Implement TypeScript types for `Story`
- [ ] Implement TypeScript types for `AcceptanceCriterion`
- [ ] Implement TypeScript types for `Task`
- [ ] Implement TypeScript types for `ArchitectureBlueprint`
- [ ] Implement TypeScript types for `ApiContract`
- [ ] Implement TypeScript types for `DataModelChange`
- [ ] Implement TypeScript types for `CodeChangeSet`
- [ ] Implement TypeScript types for `TestScenario`
- [ ] Implement TypeScript types for `AgentRunResult`
- [ ] Implement TypeScript types for `ReviewDecision`
- [ ] Implement TypeScript types for `RunEvent`
- [ ] Implement TypeScript types for `ArtifactReference`
- [ ] Implement TypeScript types for `UserIdentity`
- [ ] Implement TypeScript types for `IdempotencyKey`
- [ ] Implement TypeScript types for `ApiSubscription`
- [ ] Add contract support for `selectedAgentTypes`
- [ ] Add contract support for registry-backed `agentType` values
- [ ] Define canonical contract formats per boundary across TypeScript, JSON Schema, and API artifacts
- [ ] Decide whether `apiBoundaryVersion` and `emittedEvents` are canonical contract fields; define them in docs and schemas if retained
- [ ] Add contract support for review version and idempotent mutation metadata

### T-007 JSON Schema Package

- Outcome: Contracts can be validated consistently at runtime.
- Dependencies: `T-006`

- [ ] Create `@ai1/schemas`
- [ ] Extract schemas from [schemas.md](c:/Code/AI1-Platform/docs/schemas.md)
- [ ] Store schemas as versioned `.json` files
- [ ] Add schema export helpers
- [ ] Add schemas for run events, artifact metadata, idempotency records, and API-boundary-aware review commands
- [ ] Add schemas for SSE event envelopes and sequence-based resume parameters where required
- [ ] Verify schemas load correctly at runtime

### T-008 Runtime Validation Helpers

- Outcome: Invalid inputs and malformed outputs fail fast.
- Dependencies: `T-007`

- [ ] Add request payload validators
- [ ] Add workflow input validators
- [ ] Add agent input validators
- [ ] Add agent output validators
- [ ] Add idempotency-key and review-version validators for public mutation commands
- [ ] Enforce runtime validation for API, persistence, and artifact boundaries rather than relying on TypeScript types alone
- [ ] Add reusable validation error formatting

### T-009 SQLite Access Layer

- Outcome: Shared database access for platform services and workflow activities.
- Dependencies: `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Create `@ai1/database`
- [ ] Add SQLite client initialization
- [ ] Add migration runner
- [ ] Add transaction helpers
- [ ] Enable WAL mode during initialization
- [ ] Enable foreign key enforcement during initialization
- [ ] Add integrity-check helpers
- [ ] Add local backup or export baseline
- [ ] Add basic health and connectivity checks

### T-010 Initial SQLite Schema

- Outcome: The relational core exists and is migratable.
- Dependencies: `T-009`

- [ ] Create migrations for `stories`
- [ ] Create migrations for `acceptance_criteria`
- [ ] Create migrations for `tasks`
- [ ] Create migrations for `architecture_blueprints`
- [ ] Create migrations for `api_contracts`
- [ ] Create migrations for `data_model_changes`
- [ ] Create migrations for `agent_runs`
- [ ] Create migrations for `agent_run_artifacts`
- [ ] Create migrations for `validation_reports`
- [ ] Create migrations for `review_decisions`
- [ ] Create migrations for `run_events`
- [ ] Create migrations for `idempotency_keys`
- [ ] Create migrations for `user_identities`
- [ ] Create migrations for `api_subscriptions`
- [ ] Create migrations for `trace_links`
- [ ] Create migrations for `workspace_runs`
- [ ] Create migrations for `context_documents`
- [ ] Add storage for story-level selected agent types
- [ ] Add storage for artifact metadata including content type, size, and integrity hash
- [ ] Add storage for review version and optimistic concurrency fields
- [ ] Add storage support for retrieval provenance and context-document scoping metadata

### T-011 Repository Modules

- Outcome: Persistence logic is isolated behind typed interfaces.
- Dependencies: `T-010`

- [ ] Implement story repository
- [ ] Implement task repository
- [ ] Implement blueprint repository
- [ ] Implement runs repository
- [ ] Implement artifacts repository
- [ ] Implement reviews repository
- [ ] Implement validation repository
- [ ] Implement traceability repository
- [ ] Implement idempotency repository
- [ ] Implement run events repository
- [ ] Implement user identity repository
- [ ] Implement subscription repository for streaming consumers
- [ ] Support story reads and writes for selected agent types
- [ ] Add review-version-aware writes for approval and revision decisions
- [ ] Add repository-level tests
