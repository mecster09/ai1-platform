# 03 Workflows

## Goal

Implement Temporal workers, shared activities, and the core workflow set that orchestrates the platform.

## Target Stack

- Workflow engine: `Temporal`
- Workflow worker: `Node.js` + `TypeScript`
- Persistence: `SQLite`
- Artifact storage: filesystem
- Contract source: [temporal-workflows.md](c:/Code/AI1-Platform/docs/temporal-workflows.md)

## Tasks

### T-019 Workflow Types Package

- Outcome: Temporal-facing types are shared and versioned.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`

- [ ] Create `@ai1/workflow-types`
- [ ] Add workflow command types
- [ ] Add activity input and output types
- [ ] Add signal types
- [ ] Add task queue constants
- [ ] Add workflow ID helpers

### T-020 Workflow Worker Bootstrap

- Outcome: Temporal workers can execute workflows and activities locally.
- Dependencies: `tasks/00-foundation.md#t-004-local-infrastructure-bootstrap`, `T-019`

- [ ] Scaffold `apps/workflow-worker`
- [ ] Connect to Temporal
- [ ] Register task queues
- [ ] Register workflow definitions
- [ ] Register activity implementations
- [ ] Add common worker logging and correlation IDs

### T-021 Shared Activities

- Outcome: Reusable orchestration activity layer exists.
- Dependencies: `T-020`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`, `tasks/02-platform-api-and-web.md#t-015-artifact-storage`

- [ ] Implement `ValidateStoryInput`
- [ ] Implement `PersistStory`
- [ ] Implement `PersistAcceptanceCriteria`
- [ ] Implement `StoreArtifacts`
- [ ] Implement `CreateReviewDecision`
- [ ] Implement `UpdateStoryStatus`
- [ ] Implement `RecordAuditEntry`
- [ ] Implement `PersistAgentRunResult`
- [ ] Implement `PublishRunArtifacts`
- [ ] Implement `PersistValidationReport`
- [ ] Implement `AssembleReviewPackage`

### T-022 Create Story Workflow

- Outcome: Story creation is handled durably by Temporal.
- Dependencies: `T-021`, `tasks/02-platform-api-and-web.md#t-014-story-intake-api`

- [ ] Implement `CreateStoryWorkflow`
- [ ] Wire it to story intake API
- [ ] Persist audit entries and status transitions
- [ ] Verify idempotent replay behavior

### T-023 Decompose Story Workflow

- Outcome: The first real agent-driven stage is durably orchestrated.
- Dependencies: `T-021`, `tasks/04-context-and-workspace.md#t-030-context-service`, `tasks/05-agent-runtime-and-core-agents.md#t-037-tasks-agent-worker`

- [ ] Implement `DecomposeStoryWorkflow`
- [ ] Add clarification decision branch
- [ ] Add `ProvideClarificationSignal` handling
- [ ] Add rejection and cancellation handling
- [ ] Persist decomposition outputs and status transitions

### T-024 Create Architecture Workflow

- Outcome: Architecture generation and approval become first-class orchestrated stages.
- Dependencies: `T-021`, `tasks/04-context-and-workspace.md#t-031-retrieval-interfaces`, `tasks/05-agent-runtime-and-core-agents.md#t-038-architect-agent-worker`

- [ ] Implement `CreateArchitectureWorkflow`
- [ ] Detect approval-required changes
- [ ] Add architecture review wait state
- [ ] Add approve, revise, reject, and cancel handling
- [ ] Persist architecture artifacts and approval state

### T-025 Delivery Workflows

- Outcome: Delivery-stage workflows can fan out to the implementation agents.
- Dependencies: `T-021`, `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/04-context-and-workspace.md#t-033-workspace-isolation`, `tasks/04-context-and-workspace.md#t-034-safe-command-execution`, `tasks/06-delivery-agents.md#t-041-back-end-agent-worker`

- [ ] Implement `ImplementFrontendWorkflow`
- [ ] Implement `ImplementBackendWorkflow`
- [ ] Implement `GenerateE2ETestsWorkflow`
- [ ] Add fan-out and fan-in coordination
- [ ] Persist delivery-stage run outcomes

### T-026 Validate Delivery Workflow

- Outcome: Deterministic validation is enforced before final review.
- Dependencies: `T-021`, `tasks/05-agent-runtime-and-core-agents.md#t-036-shared-tool-adapters`, `tasks/06-delivery-agents.md#t-040-front-end-agent-worker`, `tasks/06-delivery-agents.md#t-041-back-end-agent-worker`

- [ ] Implement `ValidateDeliveryWorkflow`
- [ ] Run lint and typecheck
- [ ] Run unit tests
- [ ] Run contract validation
- [ ] Run architecture policy checks
- [ ] Run Playwright checks
- [ ] Persist validation results and blocked state

### T-027 Review And Merge Workflow

- Outcome: Human review becomes an enforced orchestration gate.
- Dependencies: `T-021`, `tasks/05-agent-runtime-and-core-agents.md#t-039-approval-integration`

- [ ] Implement `ReviewAndMergeWorkflow`
- [ ] Create final review package
- [ ] Add approve, revise, reject, and cancel handling
- [ ] Route revisions to the correct stage
- [ ] Persist final decision and completion state

### T-028 Parent Story Delivery Workflow

- Outcome: The full story lifecycle can run under one durable workflow ID.
- Dependencies: `T-022`, `T-023`, `T-024`, `T-025`, `T-026`, `T-027`

- [ ] Implement `DeliverStoryWorkflow`
- [ ] Compose stage workflows in sequence
- [ ] Compose delivery workflows in parallel
- [ ] Handle blocked and rejected terminal states
- [ ] Expose workflow queries for status and approvals
