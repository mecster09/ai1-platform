# AI1 Platform Architecture

## Purpose

This document defines the target system architecture for the AI1 Platform: a local-first software delivery platform that converts structured stories into planned, implemented, validated, and reviewable software changes through orchestrated specialist agents.

The architecture is designed around five principles:

- Workflow-driven execution instead of free-form agent chat
- Shared typed contracts instead of prompt-only coordination
- Local-first operation with optional model-provider flexibility
- Human approval at high-risk boundaries
- Deterministic validation before work is considered complete

## Architecture Summary

The platform is composed of three architectural layers:

1. Platform layer
   Hosts the UI, API, workflow orchestration, persistence, context retrieval, observability, and workspace management.
2. Delivery intelligence layer
   Defines the shared domain model, schemas, traceability model, and machine-readable artifacts that all agents use.
3. Specialist agent layer
   Executes bounded work for decomposition, architecture, implementation, and test automation using typed inputs and tool contracts.

At runtime, the platform behaves as a workflow system with controlled fan-out:

1. A user submits a story through the platform UI.
2. The platform persists the story and attachments.
3. A workflow is started in Temporal.
4. The tasks agent decomposes the story.
5. The architect agent produces the architecture blueprint and contracts.
6. The tasks breakdown selects from the preconfigured agent registry, and the chosen delivery agents execute in parallel where dependencies allow.
7. Validation gates run against generated artifacts and code changes.
8. A human reviews and approves or requests revision.
9. The platform prepares a mergeable output bundle.

## System Context

### Actors

- Product or delivery user
- Human reviewer or approver
- Platform UI
- Platform API
- Temporal workflows and workers
- Specialist agent workers
- Workspace service
- Context service
- Source repository and local worktrees
- Model provider endpoint
- Validation toolchain

### External Dependencies

The system is intentionally conservative in its dependency footprint.

- `Temporal`: workflow orchestration, retries, state progression, visibility
- `SQLite`: primary local relational data store
- `ChromaDB`: local vector similarity search on contextual artifacts, used for retrieval rather than authority
- `Git`: version control and diff generation
- `Docker`: isolated workspace execution for tool commands
- Model endpoint: `OpenAI`, `Anthropic`, local model server, or OpenAI-compatible API
- Existing project toolchain: `npm`, lint, typecheck, unit test, `Playwright`

Provider credentials and model selection should be supplied through environment configuration shared by the platform services and agent workers. Recommended variables include:

- `AI_PROVIDER`
- `AI_MODEL`
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `OPENAI_BASE_URL`

### High-Level Context Diagram

```text
+----------------+        +----------------+        +-----------------+
| Human User     | <----> | Platform Web   | <----> | Platform API    |
+----------------+        +----------------+        +--------+--------+
                                                              |
                                                              v
                                                     +--------+--------+
                                                     | Temporal        |
                                                     | Workflows       |
                                                     +---+----+----+---+
                                                         |    |    |
                              +--------------------------+    |    +--------------------------+
                              |                               |                               |
                              v                               v                               v
                    +---------+---------+           +---------+---------+           +---------+---------+
                    | Tasks Agent       |           | Architect Agent  |           | Delivery Agents    |
                    | Worker            |           | Worker           |           | Selected per story |
                    +---------+---------+           +---------+---------+           +---------+---------+
                              |                               |                               |
                              +---------------+---------------+-------------------------------+
                                              |
                                              v
                                   +----------+-----------+
                                   | Workspace Service    |
                                   | Context Service      |
                                   | Metadata + Artifacts |
                                   +----------+-----------+
                                              |
                     +------------------------+-------------------------+
                     |                        |                         |
                     v                        v                         v
               +-----+-----+            +-----+------+            +-----+------+
               | Git Repo  |            | SQLite     |            | File/Object |
               | Worktrees |            | + ChromaDB |            | Artifacts    |
               +-----------+            +------------+            +-------------+
```

## Layered Architecture

### Platform Layer

The platform layer owns all long-lived operational concerns:

- User interaction
- Workflow lifecycle
- Persistence and traceability
- Workspace isolation
- Context indexing and retrieval
- Approval and review state
- Logs, traces, and execution metadata

This layer must be stable, deterministic, and mostly agent-agnostic.

It should also centralize provider configuration loading so model credentials and model-selection settings do not have to be hard-coded inside individual workers.

#### Platform Web

`platform-web` is a local `Next.js` application that provides:

- Story submission and editing
- Decomposition and blueprint visualization
- Approval queues
- Run progress and status timelines
- Artifact browsing
- Diff comparison
- Traceability matrix views
- Error and blocker reporting

The UI should remain thin. It does not orchestrate work directly; it submits commands to the platform API and renders read models and event streams exposed by the platform boundary.

Implementation rules for `Next.js`:

- Use the App Router with Server Components by default for story, run, review, and traceability pages.
- Use Client Components only for interactive controls such as approval actions, live log viewers, diff interactions, and upload widgets.
- Use Server Actions for same-origin UI mutations when the action is initiated from `platform-web` forms or buttons and does not need to be exposed as a general-purpose HTTP contract.
- Server Actions must preserve the same validation, authorization, audit, and idempotency behavior as the typed platform boundary they sit on top of.
- Use Route Handlers only for explicit HTTP boundaries such as streaming events, artifact download or upload, webhook-style callbacks, or external tool access.
- Route Handlers are not the default internal data-access layer for server-rendered pages.
- Treat run status, logs, approvals, artifacts, and traceability screens as dynamic operational views; they must not rely on static generation.
- Define freshness per view intentionally using dynamic rendering, `no-store`, or bounded revalidation.
- Keep browser code away from direct database and workflow clients. Browser-initiated calls go through the typed platform API surface or typed Server Actions backed by that surface.
- Register framework-level telemetry through `instrumentation.ts` so request tracing and correlation IDs are captured consistently.

#### Platform API

`platform-api` is the command and query boundary of the system.

Responsibilities:

- Validate incoming story payloads
- Persist domain entities
- Upload and reference supporting artifacts
- Start, cancel, retry, and inspect workflows
- Enforce approval rules
- Expose read APIs for status, logs, and traceability
- Publish workflow commands to Temporal
- Normalize user input into typed objects

This service should be the only public application boundary. Internal workers should not be directly exposed to the UI.

The platform API must separate four public concerns:

- Command endpoints for mutations such as story creation, workflow start, approval, retry, cancellation, and revision requests
- Query endpoints for read models such as story dashboards, run summaries, review queues, and traceability
- Streaming endpoints for live workflow status, run events, and log tails
- Artifact endpoints for upload, download, diff retrieval, and evidence access

Boundary rules:

- `platform-web` may consume this boundary through HTTP or a shared typed application client, but it must not bypass validation, authorization, or audit behavior.
- Internal workers and services communicate through typed internal contracts, not the public HTTP surface.
- The public API is stable and versioned independently from internal workflow and agent contracts.

#### Workflow Engine

`Temporal` is the control plane for long-running work.

It manages:

- Workflow state
- Retries and timeouts
- Human approval waiting points
- Fan-out and fan-in coordination
- Idempotent replays
- Auditability of state transitions
- Access to the configured agent registry for dispatch decisions

Temporal workflows should encode the business process, not tool-specific logic. Agent-specific behavior belongs inside activities or worker services.

Hard rule:

- Workflow code must remain deterministic.
- Tool execution, filesystem mutation, repository mutation, external I/O, and other side effects belong in Activities or worker-side execution code.
- Approval input should be modeled through workflow signals, with status exposed through queries or read models.

#### Workspace Service

`workspace-service` manages repository access and isolated execution environments.

Responsibilities:

- Open or clone target repositories
- Create isolated working directories or worktrees
- Prepare branch naming and run metadata
- Materialize input artifacts into workspaces
- Apply generated patches
- Run build and test commands
- Capture logs, exit codes, and artifacts
- Clean up expired workspaces

Execution policy:

- Prefer structured command execution over shell-heavy execution.
- Capture `stdout`, `stderr`, exit code, duration, and execution context for each command.
- Enforce timeout, working-directory, and environment-propagation rules per command class.

Workspace isolation is a hard requirement. Multiple agents must not write to the same working directory concurrently.

#### Context Service

`context-service` provides scoped retrieval across platform knowledge.

Responsibilities:

- Index repository documentation
- Index architecture artifacts and prior runs
- Index coding standards and design system guidance
- Index API contracts and data model history
- Retrieve only relevant context for a given task and agent type
- Record which context sources were used in a run

The retrieval model must be scope-first:

- Story scope
- Repository or module scope
- Contract scope
- Historical artifact scope

Whole-repo dumping into prompts is explicitly out of scope.

Retrieval rule:

- `ChromaDB` supplies candidate context only.
- Authoritative state remains in `SQLite`, approved artifacts, and current repository contents.
- Retrieval should be filtered by explicit metadata dimensions such as story, run, agent type, artifact type, repository, module, and schema version.

#### Metadata and Artifact Stores

The platform separates structured metadata from large artifacts.

- `SQLite` stores typed domain entities, workflow state references, approvals, and traceability relations.
- `ChromaDB` stores embeddings for retrieval against indexed documents and artifacts.
- Filesystem or object-style local storage stores uploaded files, generated architecture packs, patches, logs, screenshots, and test evidence.

This separation keeps transactional data queryable while allowing large binary and generated assets to remain externalized.

Operational defaults:

- `SQLite` should run with WAL mode enabled for local read/write concurrency.
- Foreign key enforcement should be enabled explicitly.
- Large artifacts should remain outside the relational store.

### Delivery Intelligence Layer

This layer is the shared contract model that makes the platform coherent.

It contains:

- Canonical entity definitions
- JSON schemas
- Artifact types
- Traceability relations
- Review and approval metadata
- Validation policy hooks

#### Canonical Entities

Minimum core entities:

- `Product`
- `Epic`
- `Story`
- `AcceptanceCriterion`
- `Task`
- `ArchitectureBlueprint`
- `ApiContract`
- `DataModelChange`
- `CodeChangeSet`
- `TestScenario`
- `AgentRun`
- `AgentRunResult`
- `ReviewDecision`
- `Dependency`
- `Blocker`
- `Risk`

Each entity should have:

- Stable identifier
- Version
- Status
- Ownership metadata
- Source references
- Timestamps
- Relationship pointers

#### Traceability Model

Every output produced by the platform must be traceable backward and forward.

Backward traceability:

- Code change -> task -> story -> acceptance criterion
- Test scenario -> acceptance criterion -> story
- Architecture blueprint -> story and decomposition run

Forward traceability:

- Story -> tasks -> blueprint -> implementation runs -> validations -> review decisions

This model should be queryable in the UI and stored as explicit relations, not reconstructed from logs.

#### Artifact Model

Artifact types should be stored with a standard envelope:

- Artifact ID
- Artifact type
- Producing run ID
- Story ID
- Task IDs
- Format
- Path or storage URI
- Schema version
- Integrity hash

Expected artifact classes:

- `architecture.md`
- `api-contract.json`
- `data-model.json`
- `impact-analysis.json`
- `sequence-flow.md`
- Patch files
- Test reports
- Validation logs
- Review notes
- Evidence bundles

### Specialist Agent Layer

This layer contains workers with narrow responsibilities and tightly controlled I/O.

Agents are configured in advance in a platform-managed registry. Workflows and agent workers receive that registry, or an appropriately scoped view of it, as runtime context so planning and dispatch are constrained to available capabilities.

If `LangGraph` is used, it should be confined to agent-local execution inside a worker.

- `Temporal` remains the outer business-process orchestrator.
- `LangGraph` must not become a second platform-wide workflow engine.
- Agent workers that do not benefit from graph execution should remain free to use a thin custom runner instead.

Workers:

- `tasks-agent-worker`
- `architect-agent-worker`
- registered delivery-agent workers
- current built-in delivery workers:
- `frontend-agent-worker`
- `backend-agent-worker`
- `test-agent-worker`

These workers should not call each other directly. They communicate only through platform-managed artifacts and workflow state.

## Runtime Service Design

### Service Inventory

#### `platform-web`

Type:

- Interactive local web application

Primary responsibilities:

- Story intake
- Review and approvals
- Visualization of runs, artifacts, and diffs

Reads:

- Platform API query endpoints
- Platform API streaming endpoints for live run state and logs

Writes:

- Story submissions
- Approval decisions
- Retry and cancellation commands
- Artifact uploads through approved upload flows

#### `platform-api`

Type:

- Synchronous application API

Primary responsibilities:

- Command processing
- Persistence
- Workflow initiation
- Access control and approval rules

Reads:

- `SQLite`
- Artifact metadata
- Workflow state projections

Writes:

- `SQLite`
- Artifact storage metadata
- Temporal workflow start signals

#### `workflow-worker`

Type:

- Temporal worker process

Primary responsibilities:

- Execute workflow definitions
- Schedule activities
- Wait for review signals
- Aggregate run outcomes

Reads and writes:

- Temporal task queues
- Platform service activity endpoints or shared libraries

#### Agent Workers

Type:

- Temporal activity workers or adjacent execution services

Primary responsibilities:

- Consume typed work items
- Retrieve scoped context
- Operate inside isolated workspaces
- Generate structured outputs
- Publish artifacts and run results

Each worker should support:

- Retry-safe execution
- Resume from checkpoint where possible
- Explicit assumption logging
- Structured blockers
- Confidence reporting

Implementation boundary:

- Choose and document one of these models before implementation proceeds:
  - agent workers are Temporal Activity workers directly
  - agent workers are external execution services invoked by Activities
- Do not allow this boundary to remain implicit.

#### `workspace-service`

Type:

- Operational infrastructure service

Primary responsibilities:

- Repository and worktree lifecycle
- Command execution
- Log capture
- Patch application
- Cleanup and retention

#### `context-service`

Type:

- Retrieval and indexing service

Primary responsibilities:

- Embedding pipelines
- Search endpoints
- Scope filters
- Retrieval audit trails

## Workflow Architecture

### Core Workflow Set

The platform should model business flow as distinct but composable Temporal workflows.

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

These may later be composed into a parent story-delivery workflow, but they should remain independently observable and replayable.

### End-to-End Story Delivery Flow

#### 1. Story Intake

Inputs:

- Story type
- Title and narrative
- Acceptance criteria
- Priority
- Constraints
- Design references
- Target repository or module

Processing:

- Validate payload shape
- Persist normalized story
- Upload artifacts
- Generate story ID
- Start decomposition workflow

Outputs:

- Stored `Story`
- Stored `AcceptanceCriterion`
- Initial audit entry

#### 2. Decomposition

Performed by:

- `tasks-agent-worker`

Processing:

- Retrieve scoped repo context and standards
- Read the configured agent registry and available agent capabilities
- Analyze story and constraints
- Create a normalized task graph
- Flag ambiguity, blockers, and dependencies

Outputs:

- `Task[]`
- `DefinitionOfDone`
- `ImpactPrediction`
- `TaskDecompositionResult`
- Selected delivery-agent set constrained to configured availability

Gate:

- If ambiguity exceeds threshold, route to human clarification before architecture starts

#### 3. Architecture

Performed by:

- `architect-agent-worker`

Processing:

- Read story, tasks, contracts, and repo context
- Produce implementation blueprint
- Define service boundaries and integration points
- Emit machine-readable contracts

Outputs:

- `ArchitectureBlueprint`
- `ApiContract`
- `DataModelChange[]`
- `SequenceFlow`
- `ImpactAnalysis`

Gate:

- Human approval required for schema changes, new APIs, auth changes, or major refactors

#### 4. Delivery Execution

Performed by:

- one or more selected delivery-agent workers
- current built-in options are `frontend-agent-worker`, `backend-agent-worker`, and `test-agent-worker`

Processing:

- Provision isolated workspaces
- Load only relevant contracts and scoped context
- Implement changes or tests
- Publish changed files, patches, and logs

Parallelism:

- Dispatch only the delivery agents required by the task graph
- Initial supported configurations are front-end only, back-end only, test-automation only, any pair, or all three
- Front-end and back-end may run in parallel if contracts are stable
- Test generation may start once blueprint and at least interface-level outputs are available

Outputs:

- Code patches
- Test specs
- Mocks and stubs
- Run summaries
- Structured blockers

#### 5. Validation

Performed by:

- Platform validation activities through `workspace-service`

Checks:

- Lint
- Type check
- Unit tests
- Contract validation
- Architecture policy checks
- `Playwright` smoke and end-to-end execution

Outputs:

- `ValidationReport`
- Test reports
- Logs
- Failure evidence

Gate:

- Failed validation blocks review completion

#### 6. Review and Merge Preparation

Performed by:

- Human reviewer via platform UI
- Workflow signals through Temporal

Inputs:

- Diffs
- Architecture pack
- Validation evidence
- Traceability matrix
- Review checklist

Outputs:

- `ReviewDecision`
- Approved merge bundle or revision request

## Agent Architecture

### General Agent Contract

Every agent run should receive:

- A run ID
- Story ID
- Task IDs
- Agent type
- Available-agent registry view
- Allowed tools
- Input artifact references
- Context retrieval scope
- Output schema version
- Policy constraints

Every agent run should return:

- Status
- Human-readable summary
- Machine-readable result payload
- Artifact references
- Changed file list
- Assumptions
- Blockers
- Confidence score
- Review notes

### Tasks Agent Design

Primary responsibility:

- Translate a story into an executable task graph

Allowed inputs:

- Story payload
- Acceptance criteria
- Repo standards and structure
- Prior blueprint patterns if explicitly relevant

Forbidden behavior:

- Defining net-new architecture without escalation
- Inventing APIs or schema changes

Primary outputs:

- Task graph
- Dependency graph
- Risk list
- Missing information list

### Architect Agent Design

Primary responsibility:

- Produce the system change blueprint and typed contracts

Allowed inputs:

- Story
- Task decomposition
- Repo architecture and code context
- Existing API and schema definitions

Forbidden behavior:

- Writing production code as the main output
- Skipping machine-readable contract generation

Primary outputs:

- Architecture blueprint
- API contracts
- Data model changes
- Sequence flow and impact analysis

### Front-End Agent Design

Primary responsibility:

- Implement UI and client-side behavior against approved contracts

Allowed inputs:

- Front-end tasks
- Approved blueprint
- Design system context
- Existing front-end code

Forbidden behavior:

- Inventing backend APIs
- Editing files outside allowed ownership boundaries

Primary outputs:

- Patch set
- Component or integration tests
- Change log

### Back-End Agent Design

Primary responsibility:

- Implement service, API, and data-layer changes against approved architecture

Allowed inputs:

- Back-end tasks
- Approved blueprint
- Existing backend code and schema definitions

Forbidden behavior:

- Bypassing contract validation
- Introducing schema changes without approval gate satisfaction

Primary outputs:

- Patch set
- Migrations
- Tests
- Updated API contract or stubs

### Test-Automation Agent Design

Primary responsibility:

- Map acceptance criteria to executable `Playwright` coverage

Allowed inputs:

- Story and acceptance criteria
- Approved blueprint
- Front-end and back-end outputs
- Test data strategy

Forbidden behavior:

- Declaring coverage complete without criterion mapping
- Creating tests with undocumented environment assumptions

Primary outputs:

- `TestScenario[]`
- `Playwright` specs
- Evidence mapping
- Blocker reports

## Data Architecture

### Primary Datastores

#### Relational Store

`SQLite` stores:

- Stories
- Acceptance criteria
- Tasks
- Blueprints and contract metadata
- Agent runs
- Validation results
- Review decisions
- Traceability relations
- Workspace references

Recommended design approach:

- Use relational tables for source-of-truth state
- Store artifact references instead of large blobs
- Version schema-bearing entities
- Keep the initial deployment single-node and local-first; defer remote database complexity until multi-user or remote execution is needed
- Enable WAL mode and foreign key enforcement explicitly

#### Vector Store

`ChromaDB` stores embeddings for:

- Architecture docs
- Coding standards
- Prior story outputs
- API contracts
- Design system guidance
- Repository documentation

This store supports retrieval, not authority. The authority remains the structured relational model in `SQLite` and the approved artifacts on disk.

`ChromaDB` implementation must additionally define:

- collection strategy
- chunking strategy
- embedding model selection
- re-index triggers
- stale artifact cleanup behavior

#### Artifact Storage

Filesystem or local object-style storage stores:

- Uploaded reference files
- Generated markdown and JSON artifacts
- Patches
- Test evidence
- Screenshots
- Logs
- Build outputs where needed

Recommended storage layout:

```text
/artifacts/
  /stories/{storyId}/
  /runs/{runId}/
  /reviews/{reviewId}/
  /evidence/{runId}/
```

### Suggested Core Tables

Suggested initial schema:

- `stories`
- `acceptance_criteria`
- `tasks`
- `architecture_blueprints`
- `api_contracts`
- `data_model_changes`
- `agent_runs`
- `agent_run_artifacts`
- `validation_reports`
- `review_decisions`
- `trace_links`
- `workspace_runs`
- `context_documents`

### Data Lifecycle

1. Story and attachments are created.
2. Decomposition and architecture artifacts are added.
3. Agent execution artifacts accumulate under run IDs.
4. Validation reports are appended.
5. Review records finalize decision state.
6. Historical artifacts remain queryable for replay, audit, and retrieval.

## Repository and Workspace Architecture

### Isolation Strategy

The platform must isolate execution by run and by agent.

Recommended directory model:

```text
/workspaces/{storyId}/{runId}/{agentType}/
```

Each workspace should contain:

- Repository checkout or worktree
- Input artifact manifest
- Execution logs
- Generated patch files
- Validation outputs relevant to that run

### Branching Strategy

Two viable models:

1. One branch per agent run
   Best isolation, higher merge overhead.
2. One branch per story with file-level locking
   Lower merge overhead, higher coordination complexity.

Recommended MVP choice:

- One branch per agent run

It is simpler to reason about, safer for concurrency, and easier to audit.

### Workspace Execution Model

Commands should run in controlled containers or similarly isolated local workers.

Execution responsibilities:

- Install dependencies if needed
- Run targeted commands only
- Capture stdout, stderr, exit code, and timing
- Enforce timeouts
- Prevent cross-run contamination

## API Architecture

### Public Platform API

The platform API should expose resource-oriented endpoints or equivalent RPC handlers for:

- Story creation and retrieval
- Artifact upload
- Workflow start and status
- Run details and logs
- Approval actions
- Diff and traceability views
- Live run events and log streams
- Review queue and review detail reads
- Artifact download and evidence retrieval

The public API should be organized into explicit categories:

- Command API
- Query API
- Streaming API
- Artifact API

#### Boundary Between `platform-web` and `platform-api`

The separation between web and API must be implementation-specific rather than conceptual only.

- Browser-originated interactions use the public platform API boundary.
- `Next.js` Server Components and Server Actions may call a typed platform client backed by that same boundary.
- If `platform-web` and `platform-api` are colocated in the same deployment, shared server-side libraries may be used to avoid redundant loopback HTTP, but they must preserve the same validation, authorization, audit logging, and idempotency semantics as the public API.
- Route Handlers are not the default data-access layer for server-rendered pages; they exist to expose an HTTP surface where one is actually required.

#### Command API Requirements

Write endpoints must support:

- Idempotency keys for create, start, retry, cancel, approve, and revision-request operations
- Optimistic concurrency or version checks for approval and review decisions
- Audit fields for actor, timestamp, and decision rationale
- Explicit error codes for policy violations, invalid state transitions, duplicate requests, and approval-gate failures

#### Query API Requirements

Read endpoints must provide:

- Story dashboard summaries
- Review queue summaries
- Run timelines
- Validation summaries
- Traceability projections
- Artifact metadata without requiring full artifact download

#### Streaming API Requirements

Operational views need a live transport for:

- Run lifecycle events
- Workflow status changes
- Log tailing
- Approval state changes
- Validation progress

Server-Sent Events is the preferred MVP transport for local-first deployments. Polling may remain available as a fallback, but the architecture should treat streaming as the primary experience for live run views.

Streaming rules:

- SSE is server-to-browser only.
- Use named event types for distinct operational concerns.
- Include event IDs so reconnecting clients can resume safely.
- Send heartbeat comments or equivalent keep-alive behavior for long-lived streams.

#### Artifact API Requirements

Artifact endpoints must support:

- Upload initiation and completion
- Metadata lookup
- Secure local download
- Diff retrieval
- Evidence bundle retrieval
- Integrity hash reporting
- Content-type and size metadata

#### Authorization and Access Control

Even in a local-first deployment, the API must enforce action-level policy.

- Story submission, workflow start, approval, rejection, retry, and artifact access should be authorized by role or capability.
- Approval endpoints must record the approving identity and reject self-approval where policy forbids it.
- Destructive or high-risk actions require explicit policy checks before command acceptance.

#### Versioning and Idempotency

- Public API routes should be versioned at the contract level.
- Internal contracts should version independently.
- Every mutation capable of being retried by the UI, workflow engine, or network stack must accept an idempotency key.
- Streaming events should include monotonically increasing sequence numbers so reconnecting clients can resume safely.
- TypeScript types support implementation safety but do not replace runtime contract validation.

Example command set:

- `POST /stories`
- `GET /stories/{storyId}`
- `GET /stories/{storyId}/dashboard`
- `POST /stories/{storyId}/decompose`
- `POST /stories/{storyId}/architecture`
- `POST /stories/{storyId}/delivery`
- `GET /runs/{runId}`
- `GET /runs/{runId}/events`
- `GET /runs/{runId}/logs/stream`
- `GET /reviews`
- `GET /reviews/{reviewId}`
- `POST /reviews/{reviewId}/approve`
- `POST /reviews/{reviewId}/request-revision`
- `GET /artifacts/{artifactId}`
- `GET /artifacts/{artifactId}/download`

### Internal Service Contracts

Internal interactions should be typed and versioned.

Key internal request types:

- `StartWorkflowCommand`
- `AgentWorkItem`
- `ContextQuery`
- `WorkspaceProvisionRequest`
- `ValidationRequest`
- `ReviewSignal`

The internal API boundary should prioritize:

- Schema versioning
- Idempotency
- Explicit error codes
- Minimal payload ambiguity

## Validation Architecture

Validation is a first-class subsystem, not a post-processing step.

### Validation Sources

- Static analysis
- Type checking
- Unit tests
- Contract validation
- Architecture policy checks
- End-to-end automation

### Validation Policy

Every story delivery run should produce:

- Validation status per check
- Aggregate pass or fail decision
- Evidence artifacts
- Failure summaries linked to tasks and files

Validation results should be attached to the run and surfaced in the review UI before approval is allowed.

### Architecture Policy Checks

Examples:

- Front-end code does not call unapproved APIs
- Back-end changes align with approved contract versions
- Files changed match ownership and impact analysis expectations
- Schema changes have explicit approval records

## Security and Governance

### Trust Boundaries

The platform operates locally, but trust boundaries still matter.

- UI user actions are trusted only after API validation
- Agent workers are semi-trusted and must be constrained by policy
- Workspace execution is untrusted from a file-modification perspective and must be isolated
- Generated code is never trusted until validation and human review pass

### Approval Gates

Mandatory human approval points:

- New APIs
- Schema changes
- Auth or security changes
- Major architectural refactors
- Destructive edits
- Final merge bundle

### Auditability

The system must retain:

- Who submitted the story
- Which workflow and agent runs executed
- Which context documents were retrieved
- Which files were changed
- Which validations passed or failed
- Who approved or rejected outputs

## Observability Architecture

Observability should combine workflow visibility with service-level telemetry.

### Sources

- Temporal UI for workflow timelines
- Structured application logs
- Workspace command logs
- Run summaries
- Validation evidence
- Optional distributed traces for service interactions

### Required Correlation Keys

Every log and metric should include:

- `storyId`
- `runId`
- `agentType`
- `workflowId`
- `taskId` where relevant

This is necessary for root-cause analysis and historical replay.

## Deployment Architecture

### Local Development Topology

Initial local deployment:

- `platform-web`
- `platform-api`
- `workflow-worker`
- Agent workers
- `workspace-service`
- `context-service`
- `SQLite`
- `ChromaDB`
- `Temporal`
- Local artifact storage
- Local or compatible model endpoint

This can be orchestrated with local containers or a lightweight process supervisor.

### Recommended Runtime Topology

```text
+-------------------------------------------------------------+
| Local Host                                                  |
|                                                             |
|  +--------------+   +--------------+   +----------------+   |
|  | platform-web |   | platform-api |   | workflow-worker|   |
|  +--------------+   +------+-------+   +--------+-------+   |
|                             |                    |           |
|  +--------------------------+--------------------+--------+  |
|  |                Shared Platform Network                 |  |
|  +---------------+-------------------+--------------------+  |
|                  |                   |                       |
|          +-------+------+    +------+-------+               |
|          | Temporal     |    | SQLite       |               |
|          |              |    | + ChromaDB   |               |
|          +-------+------+    +------+-------+               |
|                  |                   |                       |
|   +--------------+--------+   +------+------------------+   |
|   | Agent Workers         |   | context/workspace svc   |   |
|   +--------------+--------+   +------+------------------+   |
|                  |                   |                       |
|                  +---------+---------+                       |
|                            |                                 |
|                      +-----+------+                          |
|                      | Git + FS   |                          |
|                      | Artifacts  |                          |
|                      +------------+                          |
+-------------------------------------------------------------+
```

## Failure and Recovery Model

### Failure Types

- Agent timeout
- Model provider failure
- Invalid generated contract
- Workspace execution failure
- Test environment instability
- Human rejection
- Retry exhaustion

### Recovery Strategy

- Temporal handles retries for transient failures
- Agent outputs are checkpointed as artifacts where possible
- Human rejection routes work back to the prior stage with revision context
- Workspace state is disposable and can be recreated
- Artifact and metadata persistence must survive worker restarts

### Non-Goals for MVP

- Full autonomous merge without review
- Multi-tenant distributed cloud architecture
- Large-scale event-driven infrastructure
- Free-form multi-agent discussion loops

## Implementation Priorities

### Phase 1

Build the stable control plane first:

- Platform API
- Story persistence
- Temporal workflows
- Workspace service
- Tasks and architect agents
- Approval flow

### Phase 2

Add production change generation:

- Configurable delivery-agent execution with current front-end and back-end agents
- Patch and diff handling
- Validation pipeline

### Phase 3

Add coverage and operational maturity:

- Current test-automation agent
- Traceability UI
- Context-service retrieval optimization
- Replay, observability, and policy checks

## Architectural Decisions

### Decision 1: Workflow Engine Over Agent Conversation

Reason:

- Easier auditability
- Deterministic state transitions
- Better retries and approvals
- Lower coordination ambiguity

### Decision 2: Typed Contracts Over Shared Prompt Memory

Reason:

- Better interoperability across selected agents
- Safer validation and versioning
- Less hallucinated coordination

### Decision 3: Local-First Infrastructure

Reason:

- Lower operational overhead for initial implementation
- Easier repository access
- Better control of execution isolation
- Flexible future move to hosted components if needed

### Decision 4: One Branch Per Agent Run for MVP

Reason:

- Simpler concurrency model
- Cleaner audit trail
- Safer rollback and review

## Open Design Questions

- Should the platform API own all read-model projections, or should a separate query service be introduced later?
- How much of the context service should be synchronous retrieval versus precomputed artifact bundles?
- Which contract format should be canonical first at each boundary: JSON Schema, OpenAPI, or typed TypeScript interfaces?
- Should test automation run only after implementation completion, or partially in parallel with stubs?
- What retention policy should apply to old workspaces and large evidence artifacts?

## Success Criteria

The architecture is successful if:

- A story can reliably progress from intake to reviewed output using typed workflow stages.
- Each agent consumes bounded context and produces structured artifacts.
- Validation results are machine-enforced and visible before approval.
- Every file change and test can be traced back to a story and acceptance criterion.
- Concurrent work does not corrupt repositories or cause ambiguous ownership.
