# 04 Workflows

## Goal

Implement Temporal workers, shared activities, and the core workflow set that orchestrates the platform.

## Target Stack

- Workflow engine: `Temporal`
- Workflow worker: `Node.js` + `TypeScript`
- Persistence: `SQLite`
- Artifact storage: filesystem
- Contract source: [temporal-workflows.md](c:/Code/AI1-Platform/docs/temporal-workflows.md)

## Tasks

### T-022 Workflow Types Package

- Outcome: Temporal-facing types are shared and versioned.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`

- [ ] Create `@ai1/workflow-types`
- [ ] Add workflow command types
- [ ] Add activity input and output types
- [ ] Add signal types
- [ ] Add task queue constants
- [ ] Add workflow ID helpers
- [ ] Add types for run events, idempotency validation inputs, and review version checks

### T-023 Workflow Worker Bootstrap

- Outcome: Temporal workers can execute workflows and activities locally.
- Dependencies: `tasks/00-foundation.md#t-004-local-infrastructure-bootstrap`, `T-022`, `tasks/02-agent-registry.md#t-014-agent-registry-api-and-runtime-integration`

- [ ] Scaffold `apps/workflow-worker`
- [ ] Connect to Temporal
- [ ] Register task queues
- [ ] Register workflow definitions
- [ ] Register activity implementations
- [ ] Load configured agent registry for workflow execution
- [ ] Add common worker logging and correlation IDs

### T-024 Shared Activities

- Outcome: Reusable orchestration activity layer exists.
- Dependencies: `T-023`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`, `tasks/03-platform-api-and-web.md#t-018-artifact-storage`, `tasks/02-agent-registry.md#t-013-agent-registry-persistence-and-loading`

- [ ] Implement `ValidateStoryInput`
- [ ] Implement `ValidateIdempotencyKey`
- [ ] Implement `PersistStory`
- [ ] Implement `PersistAcceptanceCriteria`
- [ ] Implement `StoreArtifacts`
- [ ] Implement `CreateReviewDecision`
- [ ] Implement `UpdateStoryStatus`
- [ ] Implement `RecordAuditEntry`
- [ ] Implement `PersistAgentRunResult`
- [ ] Implement `PublishRunEvent`
- [ ] Implement activity support for log-stream event publication
- [ ] Implement `PublishRunArtifacts`
- [ ] Implement `PersistValidationReport`
- [ ] Implement `AssembleReviewPackage`
- [ ] Implement configured agent registry lookup for workflows

### T-025 Create Story Workflow

- Outcome: Story creation is handled durably by Temporal.
- Dependencies: `T-024`, `tasks/03-platform-api-and-web.md#t-017-story-intake-api`

- [ ] Implement `CreateStoryWorkflow`
- [ ] Wire it to story intake API
- [ ] Persist audit entries and status transitions
- [ ] Verify idempotent replay behavior
- [ ] Verify public command idempotency is preserved through workflow-triggered persistence

### T-026 Decompose Story Workflow

- Outcome: The first real agent-driven stage is durably orchestrated.
- Dependencies: `T-024`, `tasks/05-context-and-workspace.md#t-033-context-service`, `tasks/06-agent-runtime-and-core-agents.md#t-040-tasks-agent-worker`, `tasks/02-agent-registry.md#t-014-agent-registry-api-and-runtime-integration`

- [ ] Implement `DecomposeStoryWorkflow`
- [ ] Add clarification decision branch
- [ ] Add `ProvideClarificationSignal` handling
- [ ] Add rejection and cancellation handling
- [ ] Pass configured available agents into decomposition
- [ ] Persist recommended and selected agent sets
- [ ] Persist decomposition outputs and status transitions

### T-027 Create Architecture Workflow

- Outcome: Architecture generation and approval become first-class orchestrated stages.
- Dependencies: `T-024`, `tasks/05-context-and-workspace.md#t-034-retrieval-interfaces`, `tasks/06-agent-runtime-and-core-agents.md#t-041-architect-agent-worker`

- [ ] Implement `CreateArchitectureWorkflow`
- [ ] Detect approval-required changes
- [ ] Add architecture review wait state
- [ ] Add approve, revise, reject, and cancel handling
- [ ] Persist architecture artifacts and approval state
- [ ] Publish run events for architecture generation and approval wait states

### T-028 Delivery Workflows

- Outcome: Delivery-stage workflows can dispatch only the selected implementation agents.
- Dependencies: `T-024`, `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/05-context-and-workspace.md#t-036-workspace-isolation`, `tasks/05-context-and-workspace.md#t-037-safe-command-execution`, `tasks/07-delivery-agents.md#t-043-front-end-agent-worker`, `tasks/07-delivery-agents.md#t-044-back-end-agent-worker`, `tasks/07-delivery-agents.md#t-045-test-automation-agent-worker`, `tasks/02-agent-registry.md#t-014-agent-registry-api-and-runtime-integration`

- [ ] Implement `ImplementFrontendWorkflow`
- [ ] Implement `ImplementBackendWorkflow`
- [ ] Implement `GenerateE2ETestsWorkflow`
- [ ] Add conditional dispatch based on selected and enabled agent types
- [ ] Add fan-out and fan-in coordination for only selected delivery workflows
- [ ] Persist delivery-stage run outcomes
- [ ] Publish delivery run lifecycle events for streaming consumers

### T-029 Validate Delivery Workflow

- Outcome: Deterministic validation is enforced before final review.
- Dependencies: `T-024`, `tasks/06-agent-runtime-and-core-agents.md#t-039-shared-tool-adapters`, `tasks/07-delivery-agents.md#t-043-front-end-agent-worker`, `tasks/07-delivery-agents.md#t-044-back-end-agent-worker`, `tasks/07-delivery-agents.md#t-045-test-automation-agent-worker`

- [ ] Implement `ValidateDeliveryWorkflow`
- [ ] Run lint and typecheck
- [ ] Run unit tests
- [ ] Run contract validation
- [ ] Run architecture policy checks
- [ ] Run Playwright checks when test-automation output exists or the selected agent set requires them
- [ ] Persist validation results and blocked state
- [ ] Publish validation progress and completion events

### T-030 Review And Merge Workflow

- Outcome: Human review becomes an enforced orchestration gate.
- Dependencies: `T-024`, `tasks/06-agent-runtime-and-core-agents.md#t-042-approval-integration`

- [ ] Implement `ReviewAndMergeWorkflow`
- [ ] Create final review package
- [ ] Add approve, revise, reject, and cancel handling
- [ ] Route revisions to the correct stage
- [ ] Persist final decision and completion state
- [ ] Require review version and idempotency validation before approval-related signals are accepted

### T-031 Parent Story Delivery Workflow

- Outcome: The full story lifecycle can run under one durable workflow ID.
- Dependencies: `T-025`, `T-026`, `T-027`, `T-028`, `T-029`, `T-030`

- [ ] Implement `DeliverStoryWorkflow`
- [ ] Compose stage workflows in sequence
- [ ] Compose only selected delivery workflows in parallel
- [ ] Handle blocked and rejected terminal states
- [ ] Expose workflow queries for status and approvals
- [ ] Ensure workflow state changes are mirrored through published run events for API streaming
