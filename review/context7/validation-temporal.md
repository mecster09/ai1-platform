# Temporal Validation

## Scope

This document validates the AI1 Platform design assumptions about `Temporal` against the official Temporal documentation surfaced through Context7.

Context7 match:

- Library ID: `/temporalio/documentation`

Validated on:

- March 11, 2026

Source basis:

- Official Temporal documentation retrieved through Context7 for workflows, activities, signals, search attributes, child workflows, and worker processes

## Validation Summary

The current documentation in `/docs` is directionally correct in choosing `Temporal` as the workflow orchestrator and durable execution layer.

The main design assumptions that are validated by the official docs are:

- `Temporal` is appropriate for long-running workflow orchestration and durable execution.
- Workflows should encode business process and orchestration.
- Side effects and tool-facing operations belong in Activities or worker-side execution code, not in workflow logic.
- Waiting for human approval through Signals is a valid and standard workflow pattern.
- Child workflows are an appropriate composition tool for decomposed business processes.
- Workers poll task queues and execute workflow or activity work.
- Search Attributes are appropriate for filtering and visibility, not as a primary business-state store.

## What the Current Docs Get Right

### 1. Temporal as the control plane

The architecture and plan documents consistently place `Temporal` at the orchestration layer for:

- workflow state
- retries
- timeouts
- approval waits
- fan-out and fan-in
- replay and auditability

This aligns with official Temporal guidance.

### 2. Workflows vs Activities separation

The docs repeatedly describe workflows as business-process orchestration and keep tool-specific or side-effecting logic in workers and execution services.

This is correct and important. Temporal documentation is explicit that workflows must remain deterministic and that interactions with external systems should be moved into Activities.

### 3. Human approval through workflow waits

The architecture uses approval gates and review signals. This is a good fit for Temporal.

Official docs confirm that workflows can wait for signals and resume when approval is received. That matches the platform’s approval model.

### 4. Child workflows for decomposition

The docs model the platform as multiple composable workflows such as:

- `CreateStoryWorkflow`
- `DecomposeStoryWorkflow`
- `CreateArchitectureWorkflow`
- delivery workflows
- `ValidateDeliveryWorkflow`
- `ReviewAndMergeWorkflow`

Temporal documentation supports child workflows as a valid composition mechanism for partitioning work and running decomposed business processes.

### 5. Task queues and worker model

The proposed architecture uses workflow workers and agent/activity workers. This is aligned with Temporal’s worker model, where workers poll task queues and execute workflow or activity tasks.

### 6. Search Attributes for visibility

The docs reference filtering, visibility, and Temporal UI correlation using workflow metadata such as story IDs and run IDs. This is consistent with Temporal Search Attributes usage.

## Corrections and Tightening Needed

### 1. Do not treat Search Attributes as a source-of-truth state model

The current docs are mostly safe here, but some phrasing could become looser over time.

Temporal docs are clear:

- Search Attributes are for filtering and visibility
- Queries expose current workflow state
- durable business state should live in workflow state and/or external stores updated through Activities

Recommendation:

- keep `SQLite` as the source of truth for platform metadata
- use Search Attributes only for workflow discoverability and operational filtering

### 2. Be explicit that tool execution should not happen in Workflow code

The docs imply this correctly, but this should be stated more bluntly anywhere workflow behavior is described.

Recommendation:

- add a short rule anywhere workflow design is discussed:
  `Temporal` workflows orchestrate; Activities and external workers execute tools, file operations, network calls, and repository mutations.

### 3. Clarify the relationship between Temporal workflows and agent workers

Some current wording mixes:

- Temporal workflows
- Temporal activity workers
- agent workers as adjacent services

That can be valid, but the implementation should choose one of two clean models:

- agent workers are Temporal Activity workers directly
- agent workers are external execution services invoked by Temporal Activities

Recommendation:

- document the chosen boundary explicitly before implementation starts

### 4. Approval signaling should define exact Temporal primitive usage

The docs mention review signals and approval waits, which is correct, but implementation design should standardize whether approvals use:

- Signals only
- Signals plus Queries
- Updates where supported and appropriate

Recommendation:

- for MVP, standardize on Signals for approval/rejection input and Queries or read models for status inspection

### 5. Workflow identity and idempotency need an implementation rule

The docs correctly emphasize idempotency, but `Temporal` implementation details need to be nailed down:

- workflow IDs
- reuse policy
- relationship between story IDs, run IDs, and workflow IDs

Recommendation:

- define a workflow ID strategy early
- define a `WorkflowIdReusePolicy` per workflow type
- ensure API idempotency keys map cleanly to workflow-start behavior

## Implementation Guidance

Based on the validation, `Temporal` remains a strong fit if implementation follows these rules:

1. Keep Workflow code deterministic.
2. Put all side effects into Activities or worker-side execution services.
3. Use Signals for human approval gates.
4. Use child workflows only where workflow decomposition improves observability or operational control.
5. Use Search Attributes for filtering in Temporal visibility tooling, not as a primary data model.
6. Keep source-of-truth platform state in `SQLite` and artifacts in external storage.
7. Define task queue ownership and workflow ID strategy before building workers.

## Result

Validation result: `pass with refinements`

`Temporal` is a valid and well-supported choice for the AI1 Platform architecture. The current design is broadly aligned with official guidance, but implementation docs should tighten the boundaries between workflows, activities, agent workers, and visibility metadata before build-out proceeds further.
