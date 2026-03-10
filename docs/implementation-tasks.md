# AI1 Platform Implementation Tasks

## Purpose

This document turns the planning set in `/docs` into a concrete implementation task backlog for building the AI1 Platform and its agents.

Target stack:

- Front end: `Next.js` + `TypeScript`
- Backend and services: `Node.js` + `TypeScript`
- Workflows: `Temporal`
- Agent runtime: `LangGraph` or a thin custom runner in `Node.js`
- Metadata store: `SQLite`
- Vector retrieval: `ChromaDB`

This backlog is aligned with:

- [plan.md](c:/Code/AI1-Platform/docs/plan.md)
- [architecture.md](c:/Code/AI1-Platform/docs/architecture.md)
- [repo-structure.md](c:/Code/AI1-Platform/docs/repo-structure.md)
- [schemas.md](c:/Code/AI1-Platform/docs/schemas.md)
- [temporal-workflows.md](c:/Code/AI1-Platform/docs/temporal-workflows.md)
- [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Delivery Principles

- Build the control plane first
- Enforce contracts before agent sophistication
- Keep all services local-first and observable
- Make every milestone runnable, not just documented
- Do not start delivery agents before workflows, schemas, and approval paths are stable

## Suggested Task Format

Each task includes:

- `ID`
- `Area`
- `Task`
- `Outcome`
- `Dependencies`

## Phase 0: Monorepo Foundation

### T-001

- Area: Workspace
- Task: Initialize the monorepo using `pnpm`, shared `TypeScript` config, linting, formatting, and root scripts.
- Outcome: A working root workspace with `apps/`, `agents/`, `packages/`, `infra/`, `scripts/`, and `docs/`.
- Dependencies: none

### T-002

- Area: Repo Structure
- Task: Create the baseline folder structure from [repo-structure.md](c:/Code/AI1-Platform/docs/repo-structure.md) for `platform-web`, `platform-api`, `workflow-worker`, `tasks-agent-worker`, `architect-agent-worker`, shared packages, and infra.
- Outcome: Source tree matches the intended monorepo shape.
- Dependencies: `T-001`

### T-003

- Area: Shared Config
- Task: Add shared `TypeScript`, ESLint, Prettier, and path alias configuration for all Node.js and Next.js packages.
- Outcome: Consistent build and lint behavior across all apps and packages.
- Dependencies: `T-001`

### T-004

- Area: Dev Environment
- Task: Create local startup scripts and container definitions for `Temporal`, `ChromaDB`, and any required support services.
- Outcome: Developers can boot the local platform infrastructure with one command.
- Dependencies: `T-001`

### T-005

- Area: Runtime State
- Task: Create runtime directories and `.gitignore` coverage for `artifacts/`, `workspaces/`, and `data/`.
- Outcome: Runtime state is isolated from source-controlled code.
- Dependencies: `T-002`

## Phase 1: Shared Contracts and Persistence

### T-006

- Area: Contracts Package
- Task: Implement `@ai1/contracts` as TypeScript domain types for `Story`, `Task`, `ArchitectureBlueprint`, `ApiContract`, `DataModelChange`, `CodeChangeSet`, `TestScenario`, `AgentRunResult`, and `ReviewDecision`.
- Outcome: All services and agents consume one typed contract layer.
- Dependencies: `T-003`

### T-007

- Area: Schemas Package
- Task: Extract the JSON schemas from [schemas.md](c:/Code/AI1-Platform/docs/schemas.md) into versioned schema files under `packages/schemas`.
- Outcome: Contracts can be validated consistently at runtime.
- Dependencies: `T-006`

### T-008

- Area: Validation
- Task: Add schema validation helpers for request payloads, workflow inputs, agent inputs, and agent outputs.
- Outcome: Invalid inputs and malformed outputs fail fast.
- Dependencies: `T-007`

### T-009

- Area: Database
- Task: Implement `@ai1/database` with SQLite connection management and migration tooling.
- Outcome: Shared database access for platform services and workflow activities.
- Dependencies: `T-003`

### T-010

- Area: Database Schema
- Task: Create the initial SQLite schema for `stories`, `acceptance_criteria`, `tasks`, `architecture_blueprints`, `api_contracts`, `data_model_changes`, `agent_runs`, `agent_run_artifacts`, `validation_reports`, `review_decisions`, `trace_links`, `workspace_runs`, and `context_documents`.
- Outcome: The relational core from [architecture.md](c:/Code/AI1-Platform/docs/architecture.md) exists and is migratable.
- Dependencies: `T-009`

### T-011

- Area: Repositories
- Task: Implement repository modules for stories, tasks, blueprints, reviews, runs, artifacts, validation reports, and traceability.
- Outcome: Persistence logic is isolated behind typed interfaces.
- Dependencies: `T-010`

## Phase 2: Platform API and Web Skeleton

### T-012

- Area: Platform API
- Task: Scaffold `apps/platform-api` in Node.js + TypeScript with health, config, logging, and error handling.
- Outcome: A production-shaped API service skeleton exists.
- Dependencies: `T-002`, `T-003`

### T-013

- Area: API Contracts
- Task: Implement the public API surface from [api-contract.json](c:/Code/AI1-Platform/docs/architecture/api-contract.json) for stories, artifacts, runs, reviews, and traceability.
- Outcome: `platform-api` exposes the planned command and query boundary.
- Dependencies: `T-012`, `T-008`, `T-011`

### T-014

- Area: Story Intake
- Task: Implement `POST /stories`, `GET /stories/{storyId}`, and `GET /stories` with schema validation and persistence.
- Outcome: Stories can be created and queried through the API.
- Dependencies: `T-013`

### T-015

- Area: Artifact Handling
- Task: Implement artifact metadata persistence and filesystem storage for story attachments and generated outputs.
- Outcome: The artifact model is usable by workflows and review UI.
- Dependencies: `T-013`, `T-011`

### T-016

- Area: Platform Web
- Task: Scaffold `apps/platform-web` with `Next.js` + `TypeScript`, shared layout, API client wiring, and status/error handling.
- Outcome: A minimal web app can communicate with `platform-api`.
- Dependencies: `T-002`, `T-003`

### T-017

- Area: Story Intake UI
- Task: Build the story submission flow with fields for title, type, narrative, acceptance criteria, constraints, priority, affected modules, and design references.
- Outcome: Users can create stories through the UI.
- Dependencies: `T-014`, `T-016`

### T-018

- Area: Story Detail UI
- Task: Build story detail and status pages showing decomposition, architecture, runs, approvals, and artifacts at a high level.
- Outcome: The web app becomes the read surface for platform progress.
- Dependencies: `T-016`, `T-014`, `T-015`

## Phase 3: Workflow Runtime

### T-019

- Area: Workflow Types
- Task: Implement `@ai1/workflow-types` for commands, activities, task queues, signals, and workflow IDs.
- Outcome: Temporal-facing types are shared and versioned.
- Dependencies: `T-006`

### T-020

- Area: Workflow Worker
- Task: Scaffold `apps/workflow-worker` in Node.js + TypeScript and register Temporal connection, workers, task queues, and common interceptors.
- Outcome: Temporal workers can execute workflows and activities locally.
- Dependencies: `T-004`, `T-019`

### T-021

- Area: Workflow Activities
- Task: Implement common activities for validation, persistence, artifact publication, review creation, audit logging, and story status updates.
- Outcome: Reusable orchestration activity layer exists.
- Dependencies: `T-020`, `T-011`, `T-015`

### T-022

- Area: Create Story Workflow
- Task: Implement `CreateStoryWorkflow` and connect it to story intake.
- Outcome: Story creation is handled durably by Temporal.
- Dependencies: `T-021`, `T-014`

### T-023

- Area: Decompose Workflow
- Task: Implement `DecomposeStoryWorkflow`, including clarification wait states and review signals.
- Outcome: The first real agent-driven stage is durably orchestrated.
- Dependencies: `T-021`, `T-030`, `T-037`

### T-024

- Area: Architecture Workflow
- Task: Implement `CreateArchitectureWorkflow`, including approval-required checks and architecture review signals.
- Outcome: Architecture generation and approval become first-class orchestrated stages.
- Dependencies: `T-021`, `T-031`, `T-038`

### T-025

- Area: Delivery Workflows
- Task: Implement `ImplementFrontendWorkflow`, `ImplementBackendWorkflow`, and `GenerateE2ETestsWorkflow`.
- Outcome: Delivery-stage workflows can fan out to the implementation agents.
- Dependencies: `T-021`, `T-032`, `T-033`, `T-034`, `T-041`

### T-026

- Area: Validation Workflow
- Task: Implement `ValidateDeliveryWorkflow` with lint, typecheck, unit, contract, architecture policy, and Playwright checks.
- Outcome: Deterministic validation is enforced before final review.
- Dependencies: `T-021`, `T-036`, `T-040`, `T-041`

### T-027

- Area: Review Workflow
- Task: Implement `ReviewAndMergeWorkflow` with final review signals, revision routing, and completion states.
- Outcome: Human review becomes an enforced orchestration gate.
- Dependencies: `T-021`, `T-039`

### T-028

- Area: Parent Orchestration
- Task: Implement `DeliverStoryWorkflow` as the parent composition over the stage workflows.
- Outcome: The full story lifecycle can run under one durable workflow ID.
- Dependencies: `T-022`, `T-023`, `T-024`, `T-025`, `T-026`, `T-027`

## Phase 4: Context and Workspace Services

### T-029

- Area: Context Package
- Task: Implement `@ai1/context-model` for document chunking, indexing metadata, and ChromaDB query helpers.
- Outcome: Shared context indexing and retrieval logic exists.
- Dependencies: `T-006`, `T-010`

### T-030

- Area: Context Service
- Task: Build `apps/context-service` for indexing repo docs, architecture docs, coding standards, API contracts, and prior story artifacts.
- Outcome: Agents can retrieve scoped context instead of full-repo dumps.
- Dependencies: `T-029`

### T-031

- Area: Retrieval API
- Task: Implement scoped retrieval endpoints or service interfaces for story scope, module scope, contract scope, and historical artifact scope.
- Outcome: Agent workflows can request relevant context deterministically.
- Dependencies: `T-030`

### T-032

- Area: Workspace Service
- Task: Build `apps/workspace-service` for repo open/clone, branch creation, isolated workspaces, command execution, and cleanup.
- Outcome: Agent execution is isolated and auditable.
- Dependencies: `T-004`, `T-011`

### T-033

- Area: Workspace Isolation
- Task: Implement one-branch-per-agent-run and one-workspace-per-run behavior under `/workspaces/{storyId}/{runId}/{agentType}`.
- Outcome: Concurrent agent execution does not corrupt repo state.
- Dependencies: `T-032`

### T-034

- Area: Command Execution
- Task: Add safe command execution with stdout/stderr capture, exit codes, timeouts, and artifactized logs.
- Outcome: Build, test, and patch application are usable by agents and validation.
- Dependencies: `T-032`

## Phase 5: Shared Agent Runtime

### T-035

- Area: Agent Runtime
- Task: Implement `@ai1/agent-runtime` with shared run envelope parsing, policy enforcement, output validation, and tool registration.
- Outcome: All agents share the same enforcement model.
- Dependencies: `T-006`, `T-007`, `T-008`

### T-036

- Area: Tool Contracts
- Task: Implement tool adapters for file reads, code search, write operations, contract reads, diff inspection, and targeted test execution.
- Outcome: Agents can work through consistent controlled tools.
- Dependencies: `T-035`, `T-034`

### T-037

- Area: Tasks Agent Worker
- Task: Build `agents/tasks-agent-worker` in Node.js + TypeScript with strict input/output contract enforcement from [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md).
- Outcome: The tasks agent can generate structured task graphs and ambiguity flags.
- Dependencies: `T-035`, `T-030`

### T-038

- Area: Architect Agent Worker
- Task: Build `agents/architect-agent-worker` to emit `ArchitectureBlueprint`, `ApiContract`, `DataModelChange[]`, sequence flow, and impact analysis artifacts.
- Outcome: The platform can produce machine-readable technical blueprints.
- Dependencies: `T-035`, `T-030`

### T-039

- Area: Review and Approval Integration
- Task: Enforce `ReviewDecision` handling between `platform-api`, `platform-web`, and Temporal signals for clarification, architecture review, and final review.
- Outcome: Human approval is part of the system, not an informal side channel.
- Dependencies: `T-013`, `T-016`, `T-020`

## Phase 6: Delivery Agents

### T-040

- Area: Front-End Agent Worker
- Task: Build `agents/frontend-agent-worker` for scoped front-end edits, contract mismatch detection, patch generation, and targeted component or integration tests.
- Outcome: The platform can generate traceable Next.js code changes from approved architecture.
- Dependencies: `T-035`, `T-032`, `T-036`

### T-041

- Area: Back-End Agent Worker
- Task: Build `agents/backend-agent-worker` for Node.js service, route, repository, and schema-adjacent changes with contract-aware output.
- Outcome: The platform can generate traceable backend changes from approved architecture.
- Dependencies: `T-035`, `T-032`, `T-036`

### T-042

- Area: Test Agent Worker
- Task: Build `agents/test-agent-worker` for Playwright scenario generation, coverage mapping, fixtures, and environment assumptions.
- Outcome: Acceptance criteria can be converted into executable end-to-end coverage.
- Dependencies: `T-035`, `T-032`, `T-036`

## Phase 7: Review, Traceability, and Validation UX

### T-043

- Area: Run Tracking
- Task: Implement API and UI support for agent runs, run summaries, logs, artifacts, and changed files.
- Outcome: Users can inspect the state and output of each run.
- Dependencies: `T-013`, `T-018`, `T-021`

### T-044

- Area: Approval UI
- Task: Build web flows for clarification requests, architecture approval, revision requests, rejection, and final review.
- Outcome: Review decisions are manageable in the product.
- Dependencies: `T-039`, `T-018`

### T-045

- Area: Traceability
- Task: Implement trace link persistence and UI views connecting story -> acceptance criteria -> tasks -> runs -> artifacts -> tests.
- Outcome: The traceability matrix promised by the plan is visible and queryable.
- Dependencies: `T-011`, `T-018`, `T-042`

### T-046

- Area: Diff and Artifact Review
- Task: Add diff viewing, artifact browsing, validation evidence views, and review package assembly screens.
- Outcome: Final review has the evidence needed for approval.
- Dependencies: `T-015`, `T-043`, `T-044`

## Phase 8: Governance and Quality Controls

### T-047

- Area: Policy Checks
- Task: Implement architecture policy checks for contract drift, unapproved API usage, file ownership violations, and unapproved schema changes.
- Outcome: Validation goes beyond tests and enforces architectural guardrails.
- Dependencies: `T-026`, `T-040`, `T-041`

### T-048

- Area: Observability
- Task: Implement shared structured logging, correlation IDs, Temporal search attributes, and run-level telemetry across services.
- Outcome: Story, run, and workflow state can be debugged end to end.
- Dependencies: `T-020`, `T-012`, `T-032`, `T-035`

### T-049

- Area: Replay and Debugging
- Task: Add replay-safe artifact lookup, workflow inspection helpers, and debugging support for blocked or rejected runs.
- Outcome: Failed workflows can be analyzed and re-driven without manual reconstruction.
- Dependencies: `T-028`, `T-048`

### T-050

- Area: Retention and Cleanup
- Task: Add retention policies and cleanup jobs for artifacts, workspaces, logs, and context indexes.
- Outcome: The local platform remains usable over time without manual maintenance.
- Dependencies: `T-032`, `T-043`, `T-048`

## Platform-Specific Task Groups

## Platform Web

- Build story intake
- Build story detail and status pages
- Build review queues and approval screens
- Build run timelines and log viewers
- Build artifact and diff viewers
- Build traceability matrix views

## Platform API

- Implement command and query endpoints
- Validate all incoming payloads against shared schemas
- Persist stories, runs, artifacts, reviews, and traceability
- Start and signal Temporal workflows
- Enforce approval and policy rules

## Workflow Worker

- Register workflows and activities
- Manage signals and queries
- Coordinate fan-out and fan-in
- Handle retries and blocked states
- Persist workflow-linked state transitions

## Context Service

- Index repository and architecture documents
- Store context metadata in SQLite
- Store embeddings and collection data in ChromaDB
- Serve scoped retrieval results to activities and agents

## Workspace Service

- Create isolated workspaces and branches
- Run commands safely
- Capture logs and evidence
- Apply patches
- Clean up expired workspaces

## Agent-Specific Task Groups

## Tasks Agent

- Parse stories and BDD criteria
- Produce normalized task graphs
- Flag ambiguity and dependency issues
- Emit handoff artifacts for architect and delivery agents

## Architect Agent

- Analyze repo architecture and task decomposition
- Produce blueprint artifacts and contracts
- Flag approval-required changes
- Emit impact analysis and sequencing guidance

## Front-End Agent

- Apply approved UI changes in Next.js and TypeScript
- Respect file ownership boundaries
- Emit contract mismatches early
- Produce component or integration tests where relevant

## Back-End Agent

- Apply approved Node.js and TypeScript service changes
- Produce migrations and tests where required
- Keep API contracts authoritative and up to date
- Emit stubs or mocks for downstream consumers

## Test-Automation Agent

- Map acceptance criteria to Playwright tests
- Generate specs, fixtures, and helpers
- Record blockers and environment assumptions
- Produce coverage and evidence mappings

## Recommended Build Order

1. Monorepo and shared packages
2. SQLite schema and repositories
3. `platform-api`
4. `platform-web`
5. `workflow-worker` plus common activities
6. `context-service`
7. `workspace-service`
8. `tasks-agent-worker`
9. `architect-agent-worker`
10. Review and approval flow
11. `frontend-agent-worker`
12. `backend-agent-worker`
13. `test-agent-worker`
14. Validation pipeline
15. Traceability, observability, and governance

## MVP Milestones

### MVP 1

- `T-001` through `T-024`
- Result: story intake, decomposition, architecture generation, and approval flow

### MVP 2

- `T-025` through `T-041`
- Result: front-end and back-end implementation agents with validation

### MVP 3

- `T-042` through `T-046`
- Result: Playwright generation, review UX, and traceability views

### MVP 4

- `T-047` through `T-050`
- Result: policy enforcement, observability, replay, and cleanup

## Success Criteria For This Backlog

- The platform can move a story from intake to reviewed output through typed stages.
- Every service and agent is implemented in `TypeScript`.
- The front end is implemented in `Next.js`.
- Backend services and agent workers are implemented in `Node.js`.
- Agent outputs are schema-validated and traceable.
- Human approval and deterministic validation remain mandatory gates.
