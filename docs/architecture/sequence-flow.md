# AI1 Platform Sequence Flow

## Purpose

This document describes the end-to-end sequence flow for story delivery in the AI1 Platform, from intake through approval and merge preparation.

The platform uses workflow orchestration, typed artifacts, and approval gates instead of free-form multi-agent conversation.

It aligns with the local-first architecture:

- `SQLite` stores structured platform state, approvals, and traceability
- `ChromaDB` supports scoped vector retrieval for context-service
- Filesystem storage holds generated artifacts, logs, patches, and evidence

## Primary Flow

### Step 1: Story Intake

1. The user creates a story in `platform-web`.
2. `platform-web` sends the story payload to `platform-api`.
3. `platform-api` validates the payload.
4. `platform-api` persists the `Story` and `AcceptanceCriterion` records in `SQLite`.
5. `platform-api` stores uploaded attachments as artifacts on the local filesystem and stores metadata in `SQLite`.
6. `platform-api` starts `CreateStoryWorkflow` and `DecomposeStoryWorkflow` in `Temporal`.
7. `Temporal` records the workflow start and queues the decomposition activity.

## Decomposition Flow

### Step 2: Tasks Agent Execution

1. `tasks-agent-worker` receives a typed `AgentWorkItem` from `Temporal`.
2. The worker requests scoped repository and standards context from `context-service`.
3. `context-service` resolves document metadata from `SQLite` and semantic retrieval candidates from `ChromaDB`.
4. The worker requests a workspace from `workspace-service` if repo inspection is required.
5. The worker analyzes:
   - Story narrative
   - Acceptance criteria
   - Constraints
   - Existing repo structure
6. The worker emits:
   - `Task[]`
   - Dependency graph
   - Definition of done
   - Risk and ambiguity flags
7. The worker stores outputs as artifacts and structured records through platform persistence.
8. `Temporal` evaluates whether ambiguity requires human clarification.

### Clarification Branch

If ambiguity is above threshold:

1. `Temporal` pauses the workflow at a review step.
2. `platform-api` creates a pending `ReviewDecision` with stage `clarification`.
3. `platform-web` shows the clarification request to a human reviewer.
4. The reviewer either:
   - Supplies missing information and resumes the workflow
   - Rejects the story for revision

If clarified successfully, the workflow resumes into architecture.

## Architecture Flow

### Step 3: Architect Agent Execution

1. `Temporal` schedules `CreateArchitectureWorkflow`.
2. `architect-agent-worker` receives:
   - Story
   - Acceptance criteria
   - Task graph
   - Relevant repository context
   - Existing contract and schema references
3. The worker generates:
   - `architecture.md`
   - `api-contract.json`
   - `data-model.json`
   - `sequence-flow.md` and, where needed, a structured flow artifact
   - `impact-analysis.json`
4. The worker persists the `ArchitectureBlueprint` and related contract artifacts.
5. `Temporal` evaluates whether approval is required.

### Architecture Approval Branch

Approval is mandatory if the blueprint includes:

- New APIs
- Schema changes
- Auth or security changes
- Major refactors
- Destructive edits

Flow:

1. `Temporal` pauses at an approval wait state.
2. `platform-api` creates a review record for the architecture stage.
3. `platform-web` shows the blueprint, contracts, impacts, and risks.
4. The reviewer either:
   - Approves the architecture
   - Requests revision
   - Rejects the story
5. Approval sends a `ReviewSignal` back to `Temporal`.
6. `Temporal` either:
   - Continues to delivery
   - Routes back to architecture revision
   - Terminates the workflow with rejection

## Delivery Flow

### Step 4: Parallel Delivery Start

Once architecture is approved:

1. `Temporal` fans out only to the delivery workflows selected from the task breakdown.
2. The initial supported delivery configurations are front-end only, back-end only, test-automation only, any pair, or all three.
3. Current built-in workflow options are:
   - `ImplementFrontendWorkflow`
   - `ImplementBackendWorkflow`
   - `GenerateE2ETestsWorkflow`
4. For each run, `workspace-service` provisions an isolated workspace:

```text
/workspaces/{storyId}/{runId}/{agentType}/
```

5. `context-service` supplies agent-scoped retrieval results using `SQLite` metadata filters and `ChromaDB` semantic search.

### Step 5: Front-End Agent

1. `frontend-agent-worker` receives:
   - Front-end tasks
   - Approved architecture blueprint
   - API contract
   - Design system guidance
2. The worker updates only allowed front-end files.
3. The worker emits:
   - Patch set
   - Changed file list
   - Component or integration tests
   - Assumptions and blockers
4. The worker stores run results and artifacts.

### Step 6: Back-End Agent

1. `backend-agent-worker` receives:
   - Back-end tasks
   - Approved blueprint
   - API contract
   - Data model changes
2. The worker updates backend services, routes, repositories, and migrations where needed.
3. The worker emits:
   - Patch set
   - Changed file list
   - Tests
   - Mocks or stubs
   - Assumptions and blockers
4. The worker stores run results and artifacts.

### Step 7: Test-Automation Agent

1. `test-agent-worker` receives:
   - Story
   - Acceptance criteria
   - Approved blueprint
   - Front-end and back-end outputs or approved interfaces
2. The worker maps each acceptance criterion to one or more `TestScenario` records.
3. The worker generates:
   - `Playwright` specs
   - Fixtures
   - Helpers
   - Evidence mapping metadata
4. The worker stores test artifacts and blockers.

## Validation Flow

### Step 8: Delivery Fan-In

1. `Temporal` waits for the selected delivery runs to complete or fail.
2. If a required run fails:
   - The workflow can retry the run
   - Or move to a blocked review state if retries are exhausted

### Step 9: Validation Execution

1. `Temporal` starts `ValidateDeliveryWorkflow`.
2. `workspace-service` or validation activities execute:
   - Lint
   - Type check
   - Unit tests
   - Contract validation
   - Architecture policy checks
   - `Playwright` smoke and end-to-end tests
3. Validation outputs are captured as:
   - `ValidationReport`
   - Test reports
   - Logs
   - Evidence artifacts
4. The validation state is persisted and linked to the story and runs.

### Validation Failure Branch

If validation fails:

1. `Temporal` marks the story as `blocked` or failed validation.
2. `platform-web` surfaces:
   - Failed checks
   - Affected files
   - Related tasks
   - Evidence links
3. A human can choose to:
   - Retry validation
   - Route back to affected delivery agents
   - Reject the delivery bundle

## Review Flow

### Step 10: Human Review

1. `Temporal` starts `ReviewAndMergeWorkflow`.
2. `platform-api` assembles the review package:
   - Diffs
   - Architecture pack
   - Validation evidence
   - Traceability matrix
   - Review checklist
3. `platform-web` presents the package to the human reviewer.
4. The reviewer can:
   - Approve
   - Request revision
   - Reject

### Step 11: Review Outcomes

If approved:

1. `ReviewDecision` is persisted as approved.
2. The workflow marks the story as completed.
3. The platform prepares a merge bundle or branch handoff.

If revision is requested:

1. `ReviewDecision` is persisted as revision requested.
2. `Temporal` routes the story back to the appropriate stage:
   - Architecture if the issue is contract or design-related
   - Delivery if the issue is implementation-related
   - Validation if only execution evidence is missing

If rejected:

1. `ReviewDecision` is persisted as rejected.
2. The story workflow is closed.

## Sequence Diagram

```text
User -> platform-web: Create story
platform-web -> platform-api: Submit story payload
platform-api -> SQLite: Save story + acceptance criteria
platform-api -> Artifact Store: Save attachments
platform-api -> Temporal: Start CreateStoryWorkflow / DecomposeStoryWorkflow

Temporal -> tasks-agent-worker: Execute decomposition
tasks-agent-worker -> context-service: Fetch scoped context
context-service -> SQLite: Load context metadata
context-service -> ChromaDB: Query semantic matches
tasks-agent-worker -> workspace-service: Provision workspace if needed
tasks-agent-worker -> SQLite: Save tasks + risks + blockers
tasks-agent-worker -> Artifact Store: Save decomposition artifacts
tasks-agent-worker -> Temporal: Return result

Temporal -> architect-agent-worker: Execute architecture generation
architect-agent-worker -> context-service: Fetch repo + contract context
context-service -> SQLite: Load context metadata
context-service -> ChromaDB: Query semantic matches
architect-agent-worker -> workspace-service: Provision architecture workspace if needed
architect-agent-worker -> SQLite: Save blueprint metadata
architect-agent-worker -> Artifact Store: Save architecture artifacts
architect-agent-worker -> Temporal: Return result

Temporal -> platform-api: Request architecture approval
platform-api -> platform-web: Show architecture review
Reviewer -> platform-web: Approve or request revision
platform-web -> platform-api: Submit review decision
platform-api -> SQLite: Save review decision
platform-api -> Temporal: Send review signal

Temporal -> frontend-agent-worker: Start FE run if selected
Temporal -> backend-agent-worker: Start BE run if selected
Temporal -> test-agent-worker: Start test generation run if selected

frontend-agent-worker -> workspace-service: Provision FE workspace
backend-agent-worker -> workspace-service: Provision BE workspace
test-agent-worker -> workspace-service: Provision test workspace

frontend-agent-worker -> Artifact Store: Save FE patch artifacts
backend-agent-worker -> Artifact Store: Save BE patch artifacts
test-agent-worker -> Artifact Store: Save Playwright artifacts

Temporal -> workspace-service: Run validation suite
workspace-service -> Artifact Store: Save logs and evidence
workspace-service -> SQLite: Save validation report

Temporal -> platform-api: Request final review
platform-api -> platform-web: Show review package
Reviewer -> platform-web: Approve / revise / reject
platform-web -> platform-api: Submit final decision
platform-api -> SQLite: Save review decision
platform-api -> Temporal: Send final review signal
Temporal -> platform-api: Mark workflow completed or reroute
```

## State Transitions

### Story Status

```text
draft
-> submitted
-> decomposed
-> architecture-pending-approval
-> in-delivery
-> validating
-> in-review
-> completed
```

Alternate transitions:

```text
architecture-pending-approval -> rejected
in-delivery -> blocked
validating -> blocked
blocked -> in-delivery
blocked -> validating
in-review -> architecture-pending-approval
in-review -> in-delivery
in-review -> rejected
```

### Agent Run Status

```text
queued -> running -> completed
queued -> running -> blocked
queued -> running -> failed
failed -> queued
blocked -> queued
completed -> needs-review
```

## Control Points and Gates

Mandatory gates:

- Ambiguity clarification after decomposition when missing information is material
- Architecture approval before delivery if high-risk changes are present
- Validation pass before final review can complete
- Final human approval before merge preparation

## Traceability Expectations

At each stage, the platform should record explicit links:

- Story -> AcceptanceCriterion
- Story -> Task
- Task -> AgentRun
- AgentRun -> Artifact
- AcceptanceCriterion -> TestScenario
- ValidationReport -> Artifact evidence
- ReviewDecision -> Story and run stage

This ensures the final review package is evidence-backed and queryable.
