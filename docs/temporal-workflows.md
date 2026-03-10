# AI1 Platform Temporal Workflows

## Purpose

This document defines the exact `Temporal` workflow structure for the AI1 Platform orchestrator, following [plan.md](c:/Code/AI1-Platform/docs/plan.md).

The workflows implement the platform's core design rule:

- Workflow-and-contract orchestration instead of free-form agent conversation
- Human approval at high-risk boundaries
- Deterministic validation before completion

The definitions here are aligned with:

- [architecture.md](c:/Code/AI1-Platform/docs/architecture.md)
- [sequence-flow.md](c:/Code/AI1-Platform/docs/architecture/sequence-flow.md)
- [schemas.md](c:/Code/AI1-Platform/docs/schemas.md)

## Workflow Set

The orchestrator owns these core workflows:

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

In practice, these should also be composed by a parent orchestration workflow:

- `DeliverStoryWorkflow`

The parent workflow is recommended even though the plan lists the stage workflows individually. It provides a single durable control plane for the whole story lifecycle while keeping each stage independently observable.

## Temporal Design Rules

- Workflows hold orchestration logic only
- Activities perform I/O, persistence, tool execution, and service calls
- Agent-specific behavior stays in workers and activities, not in workflow code
- Human approval always uses signals, never polling loops
- Retries are automatic for transient failures and explicit for policy or validation failures
- Every workflow and activity must carry `storyId`, `runId`, and stage metadata
- Workflows must have access to the configured agent registry so dispatch is constrained to enabled agents

## Task Queues

Recommended task queues:

- `platform.workflow`
- `platform.approvals`
- `platform.validation`
- `agent.tasks`
- `agent.architect`
- `agent.frontend`
- `agent.backend`
- `agent.test`
- `service.context`
- `service.workspace`

## Shared Signals

Recommended signals:

- `ApproveReviewSignal`
- `RequestRevisionSignal`
- `RejectReviewSignal`
- `ProvideClarificationSignal`
- `CancelStorySignal`
- `RetryStageSignal`

## Shared Queries

Recommended queries:

- `GetStoryStatus`
- `GetCurrentStage`
- `GetPendingApprovals`
- `GetRunSummary`
- `GetValidationState`

## Shared Activity Types

Recommended activity groups:

- Intake activities
- Persistence activities
- Context retrieval activities
- Agent dispatch activities
- Workspace activities
- Validation activities
- Review and approval activities
- Artifact publishing activities

Recommended activity names:

- `ValidateStoryInput`
- `PersistStory`
- `PersistAcceptanceCriteria`
- `StoreArtifacts`
- `CreateReviewDecision`
- `UpdateStoryStatus`
- `RecordAuditEntry`
- `FetchScopedContext`
- `DispatchTasksAgent`
- `DispatchArchitectAgent`
- `DispatchFrontendAgent`
- `DispatchBackendAgent`
- `DispatchTestAgent`
- `ProvisionWorkspace`
- `RunValidationSuite`
- `PersistValidationReport`
- `AssembleReviewPackage`
- `PublishRunArtifacts`
- `PersistAgentRunResult`

## Parent Workflow

## `DeliverStoryWorkflow`

Purpose:

- Orchestrate the entire story lifecycle from intake through final review

Inputs:

- `storyId`
- story creation command payload
- uploaded artifact references
- optional target repo or module scope
- configured agent registry reference

Outputs:

- final story status
- final review decision
- artifact bundle references
- stage run summaries

High-level flow:

1. Start `CreateStoryWorkflow`
2. Start `DecomposeStoryWorkflow`
3. If decomposition succeeds, start `CreateArchitectureWorkflow`
4. If architecture is approved, run only the delivery stage workflows selected from decomposition:
   - `ImplementFrontendWorkflow` when front-end work is required
   - `ImplementBackendWorkflow` when back-end work is required
   - `GenerateE2ETestsWorkflow` when test automation is required
   - only if that agent type is enabled in the configured registry
5. Fan in results
6. Start `ValidateDeliveryWorkflow`
7. If validation passes, start `ReviewAndMergeWorkflow`
8. Complete, reroute for revision, block, or reject

Terminal states:

- `completed`
- `blocked`
- `rejected`
- `cancelled`

## Stage Workflow Definitions

## `CreateStoryWorkflow`

Purpose:

- Normalize and persist a newly submitted story and its initial artifacts

Inputs:

- story title
- story type
- narrative
- acceptance criteria
- priority
- constraints
- affected modules
- design references
- uploaded attachments
- submitted by

Activities:

1. `ValidateStoryInput`
2. `PersistStory`
3. `PersistAcceptanceCriteria`
4. `StoreArtifacts`
5. `RecordAuditEntry`
6. `UpdateStoryStatus(submitted)`

Outputs:

- stored `Story`
- stored `AcceptanceCriterion[]`
- artifact references
- `storyId`

Failure behavior:

- Validation failure is non-retryable
- Persistence and artifact I/O failures use retry policy with backoff

Retry policy:

- initial interval: `2s`
- backoff coefficient: `2.0`
- max interval: `30s`
- max attempts: `5`

## `DecomposeStoryWorkflow`

Purpose:

- Convert a submitted story into a structured task graph and identify ambiguity

Inputs:

- `storyId`
- `Story`
- `AcceptanceCriterion[]`
- repo scope
- coding standards scope

Activities:

1. `FetchScopedContext`
2. `DispatchTasksAgent`
3. `PersistAgentRunResult`
4. `PublishRunArtifacts`
5. `UpdateStoryStatus(decomposed)`

Expected outputs:

- `Task[]`
- recommended delivery-agent selection
- dependency graph
- definition of done
- risk flags
- ambiguity flags

Selection rule:

- the tasks agent may only recommend delivery agents that are enabled in the configured registry supplied to the workflow

Decision logic:

- If ambiguity is below threshold, continue
- If ambiguity is above threshold, create clarification review and wait for signal

Clarification branch:

1. `CreateReviewDecision(reviewStage=clarification, status=pending)`
2. Wait for:
   - `ProvideClarificationSignal`
   - `RejectReviewSignal`
   - `CancelStorySignal`
3. On clarification, rerun decomposition with amended input
4. On rejection, mark story `rejected`

Retry policy:

- context and persistence activities retry
- tasks-agent dispatch retries on transient worker failure
- semantic or logical decomposition failures are returned as structured blockers, not infinite retries

## `CreateArchitectureWorkflow`

Purpose:

- Produce the approved technical blueprint and machine-readable contracts

Inputs:

- `storyId`
- `Story`
- `Task[]`
- prior context references
- decomposition artifacts

Activities:

1. `FetchScopedContext`
2. `DispatchArchitectAgent`
3. `PersistAgentRunResult`
4. `PublishRunArtifacts`
5. Determine whether approval is mandatory

Expected outputs:

- `ArchitectureBlueprint`
- `ApiContract`
- `DataModelChange[]`
- sequence flow artifact
- impact analysis artifact

Approval rules:

- approval required for:
  - schema changes
  - new APIs
  - auth or security changes
  - major refactors
  - destructive edits

Approval branch:

1. `CreateReviewDecision(reviewStage=architecture, status=pending)`
2. `UpdateStoryStatus(architecture-pending-approval)`
3. Wait for:
   - `ApproveReviewSignal`
   - `RequestRevisionSignal`
   - `RejectReviewSignal`
   - `CancelStorySignal`
4. On approve:
   - persist approval
   - continue
5. On revision:
   - rerun architecture with reviewer comments as revision context
6. On reject:
   - mark story `rejected`

Retry policy:

- agent dispatch retries for worker failures
- revision loops are explicit workflow paths, not retries

## `ImplementFrontendWorkflow`

Purpose:

- Execute front-end implementation work against approved architecture when selected by decomposition

Inputs:

- `storyId`
- approved `ArchitectureBlueprint`
- approved `ApiContract`
- front-end `Task[]`
- relevant design-system context

Activities:

1. `ProvisionWorkspace`
2. `FetchScopedContext`
3. `DispatchFrontendAgent`
4. `PersistAgentRunResult`
5. `PublishRunArtifacts`

Outputs:

- front-end patch artifacts
- changed file list
- component or integration tests
- assumptions and blockers

Failure behavior:

- worker failures retry
- contract mismatch returns blocked result
- workspace execution failure retries if transient, else blocked

## `ImplementBackendWorkflow`

Purpose:

- Execute back-end implementation work against approved architecture when selected by decomposition

Inputs:

- `storyId`
- approved `ArchitectureBlueprint`
- approved `ApiContract`
- `DataModelChange[]`
- back-end `Task[]`

Activities:

1. `ProvisionWorkspace`
2. `FetchScopedContext`
3. `DispatchBackendAgent`
4. `PersistAgentRunResult`
5. `PublishRunArtifacts`

Outputs:

- back-end patch artifacts
- changed file list
- tests
- mocks or stubs
- assumptions and blockers

Failure behavior:

- worker failures retry
- schema-risk conflicts return blocked result for human intervention

## `GenerateE2ETestsWorkflow`

Purpose:

- Generate `Playwright` coverage from approved architecture and acceptance criteria when selected by decomposition

Inputs:

- `storyId`
- `Story`
- `AcceptanceCriterion[]`
- approved `ArchitectureBlueprint`
- available implementation outputs or interface stubs

Activities:

1. `ProvisionWorkspace`
2. `FetchScopedContext`
3. `DispatchTestAgent`
4. `PersistAgentRunResult`
5. `PublishRunArtifacts`

Outputs:

- `TestScenario[]`
- `Playwright` specs
- fixtures and helpers
- coverage mapping
- blockers

Failure behavior:

- if interface inputs are insufficient, return blocked with explicit assumptions
- retry only transient worker and persistence failures

## `ValidateDeliveryWorkflow`

Purpose:

- Run deterministic checks across generated code and test outputs

Inputs:

- `storyId`
- delivery run IDs
- workspace references
- approved architecture artifacts

Activities:

1. `RunValidationSuite`
2. `PersistValidationReport`
3. `PublishRunArtifacts`
4. `UpdateStoryStatus(validating)`

Validation checks:

- lint
- type check
- unit tests
- contract validation
- architecture policy checks
- `Playwright` smoke tests
- `Playwright` end-to-end tests

Outputs:

- `ValidationReport`
- logs
- test evidence
- failure evidence

Failure behavior:

- infrastructure failures retry
- deterministic validation failures do not retry automatically
- on validation failure:
  - `UpdateStoryStatus(blocked)`
  - expose evidence for human review or reroute

Retry policy:

- initial interval: `5s`
- backoff coefficient: `2.0`
- max interval: `1m`
- max attempts: `3`

## `ReviewAndMergeWorkflow`

Purpose:

- Coordinate final human review and either completion or reroute for revisions

Inputs:

- `storyId`
- review package inputs
- diff artifacts
- architecture artifacts
- validation report
- traceability matrix

Activities:

1. `AssembleReviewPackage`
2. `CreateReviewDecision(reviewStage=final-review, status=pending)`
3. `UpdateStoryStatus(in-review)`

Signals awaited:

- `ApproveReviewSignal`
- `RequestRevisionSignal`
- `RejectReviewSignal`
- `CancelStorySignal`

On approve:

1. persist approval decision
2. mark story `completed`
3. emit merge bundle or branch handoff artifact

On request revision:

1. persist revision decision
2. route back based on revision scope:
   - architecture issue -> `CreateArchitectureWorkflow`
   - implementation issue -> delivery workflows
   - evidence-only issue -> `ValidateDeliveryWorkflow`

On reject:

1. persist rejection
2. mark story `rejected`

## Child Workflow Composition

Recommended parent-to-child composition inside `DeliverStoryWorkflow`:

```text
CreateStoryWorkflow
-> DecomposeStoryWorkflow
-> CreateArchitectureWorkflow
-> [selected delivery workflows] in parallel
-> ValidateDeliveryWorkflow
-> ReviewAndMergeWorkflow
```

Fan-out rules:

- only selected delivery workflows are started
- initial supported selections are frontend only, backend only, test-automation only, any pair, or all three
- front-end and back-end can run in parallel after architecture approval
- test generation can run in parallel once approved contracts exist
- delivery fan-in waits for all selected child workflows

## State Mapping

Recommended story state ownership by workflow:

- `CreateStoryWorkflow` -> `submitted`
- `DecomposeStoryWorkflow` -> `decomposed`
- `CreateArchitectureWorkflow` -> `architecture-pending-approval`
- delivery workflows -> `in-delivery`
- `ValidateDeliveryWorkflow` -> `validating` or `blocked`
- `ReviewAndMergeWorkflow` -> `in-review`, `completed`, or `rejected`

## Review and Signal Model

All human approvals should follow the same pattern:

1. Workflow creates a `ReviewDecision` record through an activity
2. UI presents pending review
3. Reviewer acts in `platform-web`
4. `platform-api` validates the decision and signals the workflow
5. Workflow resumes from the exact wait point

This applies to:

- decomposition clarification
- architecture approval
- final review

## Idempotency Requirements

Activities must be idempotent for workflow replay safety.

Required idempotency keys:

- `storyId`
- `runId`
- `workflowId`
- `reviewId` where relevant
- `artifactId` for publish operations

Examples:

- `PersistStory` should upsert by `storyId`
- `CreateReviewDecision` should deduplicate by `workflowId + reviewStage`
- `PersistValidationReport` should deduplicate by `storyId + validation run`

## Failure and Recovery Rules

Transient failures:

- network or service startup issues
- temporary workspace provisioning issues
- temporary model endpoint failures
- temporary lock contention

Handling:

- Temporal retries with bounded backoff

Non-transient failures:

- invalid story payload
- schema contract mismatch
- blocked approval decision
- deterministic test failure
- policy violation

Handling:

- return structured failure payload
- persist blockers and evidence
- move workflow to blocked, rejected, or revision path

## Suggested Workflow IDs

Recommended workflow ID shapes:

- `story/{storyId}/deliver`
- `story/{storyId}/create`
- `story/{storyId}/decompose`
- `story/{storyId}/architecture`
- `story/{storyId}/frontend/{runId}`
- `story/{storyId}/backend/{runId}`
- `story/{storyId}/test/{runId}`
- `story/{storyId}/validate/{runId}`
- `story/{storyId}/review/{runId}`

## Suggested Search Attributes

Recommended Temporal search attributes:

- `storyId`
- `runId`
- `agentType`
- `workflowStage`
- `storyStatus`
- `reviewStage`
- `priority`
- `repoScope`

These support filtering in `Temporal UI` and correlate workflow history with platform state in `SQLite`.

## MVP Implementation Order

Following the roadmap in [plan.md](c:/Code/AI1-Platform/docs/plan.md):

1. Implement `CreateStoryWorkflow`
2. Implement `DecomposeStoryWorkflow`
3. Implement `CreateArchitectureWorkflow`
4. Add architecture approval signal handling
5. Add delivery workflows
6. Add `ValidateDeliveryWorkflow`
7. Add `ReviewAndMergeWorkflow`
8. Add `DeliverStoryWorkflow` orchestration wrapper if not built first

## Why This Fits The Plan

- It keeps `Temporal` as the durable orchestration layer.
- It encodes the workflow-and-contract model explicitly.
- It separates orchestration from agent execution.
- It preserves human approval gates at architectural and review boundaries.
- It supports deterministic validation before completion.
