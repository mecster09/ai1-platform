# Tasks Remediation

## Purpose

This document consolidates the gaps, contradictions, and improvement actions identified across the files in `/tasks` when validated against the current contents of:

- `/docs/plan.md`
- `/docs/architecture.md`
- `/docs/tech-stack.md`
- `/docs/temporal-workflows.md`
- `/docs/architecture/api-contract.json`
- `/docs/architecture/data-model.json`
- `/docs/architecture/sequence-flow.md`

It is intended to guide updates to the task checklists before implementation proceeds further.

## Priority Order

Recommended remediation order:

1. Resolve task dependency cycles and workflow entrypoint inconsistencies
2. Lock the `Temporal` and `LangGraph` boundary
3. Tighten workspace execution and retrieval tasks
4. Tighten API/SSE and contract-boundary tasks
5. Add missing persistence, evidence, and governance tasks

## 1. Dependency Graph And Workflow Entry

### Problem

The current task dependency graph contains a bootstrap cycle:

- `T-014` depends on `T-015` and `T-023`
- `T-016` depends on `T-014`
- `T-023` depends on `T-014`

This makes agent-registry integration impossible to complete in the current order.

There is also an orchestration mismatch:

- `T-025` still implies the API wires directly to `CreateStoryWorkflow`
- the current architecture and sequence flow now expect the API to start `DeliverStoryWorkflow`

### Affected Files

- `/tasks/02-agent-registry.md`
- `/tasks/03-platform-api-and-web.md`
- `/tasks/04-workflows.md`

### Required Remediation

- Split `T-014` into two tasks:
  - pre-runtime registry contracts/loading integration
  - later API and workflow integration
- Remove the cycle between `T-014`, `T-016`, and `T-023`
- Update `T-025` so story intake starts `DeliverStoryWorkflow`
- Clarify whether `CreateStoryWorkflow` remains an internal child workflow only

## 2. Temporal And LangGraph Boundary

### Problem

The updated architecture now requires:

- `Temporal` to own story-level orchestration, retries, approval waits, and multi-agent coordination
- `LangGraph` to be optional and agent-local only

The current task files still leave this boundary implicit.

### Affected Files

- `/tasks/04-workflows.md`
- `/tasks/06-agent-runtime-and-core-agents.md`
- `/tasks/07-delivery-agents.md`

### Required Remediation

- Add a task to explicitly choose and document:
  - which agents use `LangGraph`
  - which agents use a thin custom runner
- Add a task that explicitly forbids `LangGraph` from becoming a second platform-wide orchestrator
- Add a task that defines whether agent workers are:
  - Temporal Activity workers directly
  - adjacent execution services invoked by Activities
- Ensure worker/runtime tasks say that approval flow, retries, and story-level orchestration stay in `Temporal`

## 3. Safe Command Execution

### Problem

The current workspace task list captures output and timeouts, but it does not yet encode the execution controls now required by the architecture.

Missing controls include:

- structured execution over shell-heavy invocation
- working-directory enforcement
- environment propagation rules
- command classification and policy checks
- execution metadata capture beyond basic stdout/stderr

### Affected Files

- `/tasks/05-context-and-workspace.md`

### Required Remediation

- Expand `T-037` to include:
  - prefer structured execution semantics such as `spawn` or `execFile`
  - avoid default shell-based execution
  - enforce working directory per run
  - define environment variable allowlist/propagation rules
  - capture command name, args, cwd, duration, exit code, and timeout outcome
  - distinguish validation commands from agent tool commands

## 4. Retrieval And ChromaDB Scope

### Problem

The current context tasks are too general compared with the updated architecture and validation guidance.

Missing elements:

- collection strategy
- metadata scoping fields
- retrieval provenance
- explicit non-authoritative behavior
- indexing lifecycle details

### Affected Files

- `/tasks/05-context-and-workspace.md`

### Required Remediation

- Expand `T-032`, `T-033`, and `T-034` to include:
  - collection strategy selection
  - metadata dimensions such as:
    - `storyId`
    - `runId`
    - `agentType`
    - `artifactType`
    - `repo`
    - `module`
    - `schemaVersion`
  - explicit rule that `ChromaDB` provides retrieval candidates only
  - retrieval provenance recording per run
  - chunking strategy
  - re-index triggers
  - stale embedding cleanup

## 5. TypeScript And Runtime Validation Boundary

### Problem

The task files correctly emphasize typed contracts, but they do not fully encode the rule that `TypeScript` is compile-time only and cannot replace runtime validation.

There are also some task-level fields that appear speculative and are not clearly defined in the canonical docs.

### Affected Files

- `/tasks/01-contracts-and-persistence.md`
- `/tasks/06-agent-runtime-and-core-agents.md`

### Required Remediation

- Add an explicit task stating that public API, persistence, and artifact boundaries require runtime validation through schemas
- Define which contract artifacts are canonical at which boundaries
- Review these fields and either:
  - define them in canonical docs and schemas
  - or remove them from the tasks

Fields that need resolution:

- `apiBoundaryVersion`
- `emittedEvents`
- `mustUseApprovedApiBoundary`
- `requiresIdempotentMutations`
- `requiresReviewVersionCheck`

## 6. SQLite Operational Defaults

### Problem

The updated architecture now makes SQLite operational defaults explicit, but the tasks do not.

Missing defaults:

- WAL mode
- foreign key enforcement
- basic integrity/backup expectations

### Affected Files

- `/tasks/01-contracts-and-persistence.md`

### Required Remediation

- Add tasks to `T-009` and/or `T-010` for:
  - enabling WAL mode
  - enabling foreign key enforcement
  - adding integrity-check support
  - defining a local backup/export baseline

## 7. SSE Contract And Live Views

### Problem

The updated API contract and architecture now define stronger SSE behavior than the task files currently encode.

Missing items:

- named event taxonomy
- sequence-based resume
- heartbeat behavior
- explicit separation of `run.log` from other event types
- reconnect-state reconciliation in the UI

### Affected Files

- `/tasks/03-platform-api-and-web.md`
- `/tasks/08-review-traceability-and-validation-ux.md`

### Required Remediation

- Expand `T-016` and `T-019` to include:
  - named event support
  - `afterSequence` handling
  - heartbeat/keep-alive behavior
  - ordered reconnect behavior
- Expand `T-046` to include:
  - resume from last seen sequence
  - reconcile live state after reconnect
  - distinguish logs from non-log events in the client model

## 8. Review, Approval, And Idempotency

### Problem

The review and workflow tasks cover basic approval paths, but some updated invariants are still not explicit enough in the task layer.

Missing emphasis:

- Signals as the workflow approval input mechanism
- Queries or read models for status
- consistent idempotency semantics across commands and workflow starts

### Affected Files

- `/tasks/03-platform-api-and-web.md`
- `/tasks/04-workflows.md`
- `/tasks/06-agent-runtime-and-core-agents.md`

### Required Remediation

- Ensure task wording explicitly uses:
  - workflow signals for approval/resume input
  - read/query surfaces for status inspection
- Add a task to define workflow ID strategy and reuse policy
- Add a task to define mapping between:
  - API idempotency keys
  - story IDs
  - workflow IDs
  - run IDs

## 9. Delivery-Agent Evidence And Traceability

### Problem

The delivery-agent tasks generate artifacts and tests, but some evidence requirements are still implicit.

Missing items:

- trace/report/screenshot policy for test automation outputs
- explicit traceability links from acceptance criteria to test evidence
- structured evidence outputs for review

### Affected Files

- `/tasks/07-delivery-agents.md`
- `/tasks/08-review-traceability-and-validation-ux.md`

### Required Remediation

- Expand `T-045` to include:
  - evidence artifact generation policy
  - trace or screenshot output policy where appropriate
  - explicit acceptance-criterion-to-evidence linkage
- Expand `T-048` and `T-049` to ensure review surfaces can navigate from:
  - acceptance criterion
  - to test scenario
  - to evidence artifact

## 10. Governance And Retrieval Provenance

### Problem

The governance tasks do not yet enforce the retrieval audit expectations introduced in the updated architecture.

### Affected Files

- `/tasks/09-governance-and-operations.md`

### Required Remediation

- Expand `T-050` to include checks for:
  - missing retrieval provenance
  - missing context-document audit trails
  - use of undocumented retrieval scopes
- Expand `T-051` to include telemetry for:
  - retrieval source counts
  - retrieval filter scopes
  - SSE reconnect and resume behavior

## 11. Foundation And Shared Config

### Problem

The foundation tasks set up infra and env templates, but they do not yet define configuration ownership clearly enough.

### Affected Files

- `/tasks/00-foundation.md`

### Required Remediation

- Expand `T-004` to define:
  - which services consume which environment variables
  - how shared config is loaded across services and workers
  - which values are runtime-only vs repo-defaulted

## Suggested Task File Update Order

1. `/tasks/04-workflows.md`
2. `/tasks/06-agent-runtime-and-core-agents.md`
3. `/tasks/05-context-and-workspace.md`
4. `/tasks/03-platform-api-and-web.md`
5. `/tasks/01-contracts-and-persistence.md`
6. `/tasks/02-agent-registry.md`
7. `/tasks/07-delivery-agents.md`
8. `/tasks/08-review-traceability-and-validation-ux.md`
9. `/tasks/09-governance-and-operations.md`
10. `/tasks/00-foundation.md`

## Next Step

After updating the task files, re-review the following for alignment:

- `/tasks/README.md`
- `/docs/plan.md`
- `/docs/architecture.md`
- `/docs/temporal-workflows.md`
- `/docs/architecture/api-contract.json`
- `/docs/architecture/data-model.json`
