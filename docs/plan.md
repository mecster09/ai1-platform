# AI1 Platform Plan

## Overview

You are building a local software delivery platform with:

- A shared workflow engine
- A shared project memory and context layer
- A shared repo and workspace model
- A human approval layer
- Specialized agents sitting on top

A strong local-first implementation is:

- `Temporal` for durable orchestration, retries, state, and visibility
- `LangGraph` or a thin custom runner for controlled agent execution
- Local model support through an OpenAI-compatible endpoint or equivalent model clients

## Target Architecture

Build the platform in three layers.

### 1. Platform Layer

This is the local platform that manages all work.

Core services:

- Workflow orchestrator: `Temporal`
- Agent execution runtime: `LangGraph` or a thin custom agent runner
- Artifact store: filesystem + Git repo + object storage folder
- Metadata store: `SQLite`
- Vector and context store: `ChromaDB`
- Queue and events: let `Temporal` cover most of this; avoid adding Kafka early
- Web UI: `Next.js` app for story intake, approvals, run status, logs, and artifacts
- Observability: `Temporal UI` + structured logs + traces

### 2. Delivery Intelligence Layer

This is where the shared domain model lives.

Key entities:

- Product
- Epic
- Story
- Acceptance Criterion
- Design Change
- Architecture Blueprint
- Task
- Agent Run
- Code Change
- Test Suite
- Review Decision
- Dependency / Blocker / Risk

All agents should operate on the same structured objects, not loose prompts.

### 3. Specialist Agent Layer

The platform always includes two control agents:

- Tasks agent
- Architect agent

The initial delivery-agent registry includes:

- Front-end agent
- Back-end agent
- Test-automation agent

Agents should be configured in advance in a platform-managed agent registry and made available to workflows and agent workers as part of runtime context.

Delivery-agent selection is story-specific. The tasks agent determines which delivery agents are required from the task breakdown, so a story may run:

- Front-end only
- Back-end only
- Test-automation only
- Any combination of two of the above
- All three

Future delivery agents can be added later without changing the control-plane model.

Each agent should be:

- Narrow in responsibility
- Deterministic where possible
- Tool-driven
- Constrained by schemas and contracts

## Agent Operating Model

### Tasks Agent

Purpose: Convert story inputs into executable work packages.

Inputs:

- User story or technical story
- BDD acceptance criteria
- Design changes, mockups, or notes
- Existing architecture blueprint where available
- Repo context and coding standards

Outputs:

- Normalized story record
- Task breakdown by area
- Recommended delivery-agent selection
- Selection constrained to agents available in the configured registry
- Dependencies
- Definition of done
- Predicted impacted files and modules
- Risk flags and ambiguity flags
- Handoff packet to architect and implementation agents

Rules:

- Do not invent solution design beyond basic decomposition
- Identify unknowns and request architecture input when needed

### Architect Agent

Purpose: Produce the technical blueprint.

Inputs:

- Story
- Acceptance criteria
- Task decomposition
- Existing repo architecture
- Current APIs and data model

Outputs:

- Solution design
- Component and service boundaries
- API contract
- Data flow
- Data model changes
- Validation, security, and performance notes
- Implementation sequencing
- Testing strategy hooks

The architect agent should produce machine-readable artifacts, not only Markdown, for example:

- `architecture.md`
- `api-contract.json`
- `data-model.json`
- `sequence-flow.json`
- `impact-analysis.json`

### Front-end Agent

Purpose: Build `React` / `Next.js` / `TypeScript` changes.

Inputs:

- Front-end tasks from the tasks agent
- Blueprint from the architect
- Design system guidance
- Existing codebase context

Outputs:

- Code changes
- Component tests where relevant
- Route, page, and component updates
- API integration code
- PR summary or change log

Constraints:

- Must not invent backend APIs that conflict with the architect contract
- Must emit contract mismatches early
- Must follow file ownership boundaries

### Back-end Agent

Purpose: Build `Node.js` / `TypeScript` changes.

Inputs:

- Back-end tasks
- Architecture blueprint
- API and data model design
- Existing service code context

Outputs:

- Route, service, and repository changes
- Migrations where required
- Validation and auth logic
- Contract tests or unit tests
- API docs updates

Constraints:

- Should own source-of-truth API contract generation where relevant
- Should produce mocks and stubs for front-end and test agents

### Test-Automation Agent

Purpose: Produce `Playwright` end-to-end coverage.

Inputs:

- Story
- BDD acceptance criteria
- Architecture blueprint
- Front-end and back-end outputs
- Seeded test data strategy

Outputs:

- `Playwright` specs
- Page objects and helpers
- Test fixtures
- Environment assumptions
- Coverage mapping back to acceptance criteria

Rules:

- Map each BDD scenario to a test case ID
- Track automation status
- Record blockers
- Record evidence paths

## Most Important Design Decision

Do not let agents talk freely to each other in long natural-language conversations.

Use a workflow-and-contract model instead:

1. Story submitted
2. Tasks agent produces a structured breakdown
3. Architect agent produces the blueprint
4. Only the delivery agents selected by the tasks breakdown execute, in parallel where possible
5. Validation and review gates run
6. Human approves or requests revisions
7. Merge or package the output

This is more stable than open-ended agent-to-agent chat.

## Proposed Workflow

### Phase A: Intake

User enters:

- Story type: user or technical
- Title
- Narrative
- BDD acceptance criteria
- Design references
- Priority
- Constraints
- Affected app or module

Platform actions:

- Validate required fields
- Store raw and normalized story
- Attach supporting artifacts

### Phase B: Decomposition

Tasks agent:

- Parses the story
- Identifies workstreams
- Creates tasks
- Flags ambiguity and missing information

### Phase C: Architecture

Architect agent:

- Analyzes the repo and task pack
- Generates the blueprint
- Emits contracts and the impact map

### Phase D: Delivery

Run in parallel where possible:

- Dispatch only the selected delivery agents for the story
- Current built-in options are front-end, back-end, and test-automation
- Valid initial configurations include one agent, any pair, or all three

### Phase E: Validation

Automated checks:

- Lint
- Type check
- Unit tests
- Contract validation
- `Playwright` smoke and end-to-end tests
- Architecture policy checks

### Phase F: Review

Human-in-the-loop approvals:

- Approve architecture
- Approve code changes
- Approve test coverage
- Approve the final merge bundle

## Shared Data Contracts

Define these JSON schemas before writing agents:

- Story
- AcceptanceCriterion
- Task
- ArchitectureBlueprint
- ApiContract
- DataModelChange
- CodeChangeSet
- TestScenario
- AgentRunResult
- ReviewDecision

This is where most platforms succeed or fail.

Example story shape:

```json
{
  "storyId": "ST-1024",
  "title": "User can reset password",
  "type": "user-story",
  "bddScenarios": [
    {
      "id": "AC-1",
      "given": "a registered user is on the login page",
      "when": "they request a password reset",
      "then": "a reset email is sent"
    }
  ],
  "designArtifacts": [],
  "constraints": ["must use existing auth service"],
  "priority": "high"
}
```

Example task output:

```json
{
  "storyId": "ST-1024",
  "recommendedAgentTypes": ["frontend", "backend", "test-automation"],
  "tasks": [
    {
      "id": "FE-1",
      "agentType": "frontend",
      "title": "Add forgot password form",
      "dependsOn": []
    },
    {
      "id": "BE-1",
      "agentType": "backend",
      "title": "Create password reset request endpoint",
      "dependsOn": []
    },
    {
      "id": "TA-1",
      "agentType": "test-automation",
      "title": "Create Playwright scenario for reset request flow",
      "dependsOn": ["FE-1", "BE-1"]
    }
  ]
}
```

## Local Platform Components

### 1. UI Portal

A local `Next.js` app with pages for:

- Create story
- View decomposition
- View architecture blueprint
- Trigger agent runs
- Inspect logs and artifacts
- Approve or reject outputs
- Compare generated diffs
- View the traceability matrix

### 2. Orchestrator Service

A `Node.js` service that:

- Receives platform commands
- Starts `Temporal` workflows
- Coordinates agent runs
- Loads the configured agent registry into workflow context
- Writes state transitions

### 3. Agent Runner Services

Separate local services or workers:

- `tasks-agent-worker`
- `architect-agent-worker`
- delivery-agent workers selected per story
- initial delivery workers:
- `frontend-agent-worker`
- `backend-agent-worker`
- `test-agent-worker`

Each worker:

- Receives a typed work item
- Receives the configured agent registry or relevant filtered view of it
- Loads only relevant context
- Executes tools
- Writes structured outputs

### 4. Repo Workspace Manager

Responsibilities:

- Clone or open local repos
- Create branches
- Isolate workspaces per run
- Apply diffs
- Run `npm`, test, and build commands
- Capture outputs safely

Use isolated work directories per run:

```text
/workspaces/{storyId}/{runId}/{agentType}/
```

### 5. Context Service

Responsible for:

- Embedding and indexing repo docs
- Architecture docs
- Coding standards
- Prior stories
- API contracts
- Design tokens and component library information

Use scoped retrieval rather than whole-repo dumping.

## How Agents Should Work

Each agent should have:

### A. System Contract

Defines:

- Role
- Allowed tools
- Required outputs
- Forbidden actions
- Quality bar

### B. Tooling Contract

Examples:

- Read files
- Search code
- Write files
- Run tests
- Run linters
- Inspect `package.json`
- Read schemas and contracts
- View Git diffs
- Create patches

### C. Output Contract

Every run must return:

- Status
- Summary
- Structured artifact paths
- Changed files
- Assumptions
- Blockers
- Confidence
- Review notes

## Critical Guardrails

1. Human approval at architectural boundaries
   Require approval before schema changes, new APIs, auth or security changes, major refactors, and destructive file edits.
2. Branch and sandbox isolation
   Never let multiple agents write directly to the same branch or worktree simultaneously.
3. Contracts over prose
   Front-end and back-end coordination should happen via `OpenAPI`, JSON Schema, typed interfaces, or event contracts.
4. Traceability
   Every output should trace back to story ID, task ID, and acceptance criterion ID.
5. Deterministic validation
   Never trust "the agent says it is done"; gate on builds, tests, contract validation, lint, type checks, and a review checklist.

## Suggested MVP Roadmap

### MVP 1: Platform Skeleton

Build:

- Story intake UI
- Database schemas
- `Temporal` workflow
- Single repo workspace manager
- Tasks agent and architect agent only
- Human approval step

Goal:

- Convert a story into a task pack and architecture pack

### MVP 2: Delivery Agents

Add:

- Configurable delivery-agent selection from the tasks breakdown
- Front-end agent
- Back-end agent
- File patch generation
- Local Git branch handling
- Automated lint, build, and test execution

Goal:

- Generate code changes per story

### MVP 3: Test Automation

Add:

- `Playwright` generation
- Acceptance-criteria-to-test mapping
- Test environment seeding
- Execution reporting

Goal:

- Produce runnable end-to-end coverage

### MVP 4: Governance and Quality

Add:

- Approval workflows
- Policy checks
- Observability dashboards
- Replay and debugging
- Reuse of prior artifacts

Goal:

- Make the platform dependable, not just clever

## Recommended Tech Stack

For the stated stack, this is a strong fit:

- Platform UI: `Next.js` + `TypeScript`
- API and platform service: `Node.js` + `TypeScript`
- Workflow orchestration: `Temporal` self-hosted or local
- Agent runtime: `LangGraph`
- DB: `SQLite`
- Vector search: `ChromaDB`
- Workspace execution: Docker-based isolated workers
- Source control: Git
- Testing: `Playwright` + existing lint, typecheck, and unit frameworks
- Model access: local model endpoint or OpenAI-compatible provider abstraction

Do not start with:

- Too many agent frameworks
- Free-form agent chat
- Event buses
- Kubernetes
- Full autonomy

## Practical System Decomposition

Services:

- `platform-web`
- `platform-api`
- `workflow-worker`
- `tasks-agent-worker`
- `architect-agent-worker`
- `frontend-agent-worker`
- `backend-agent-worker`
- `test-agent-worker`
- `context-service`
- `workspace-service`

Core workflows:

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

## Success Criteria

### Tasks Agent

- Tasks are complete, non-overlapping, and dependency-aware
- Every acceptance criterion is covered
- Ambiguities are surfaced rather than hidden

### Architect Agent

- Blueprint is implementable
- Contracts are explicit
- Data and API impacts are clear

### Delivery Agents Present For The Story

- Code compiles
- Changes adhere to the existing architecture
- Change set is minimal and traceable

### Test Agent

- Every BDD criterion maps to at least one test
- Flaky assumptions are documented
- Tests run locally

## Biggest Risks

- Context overload: solve with scoped retrieval and structured artifacts
- Agent conflicts in the repo: solve with branch or worktree isolation and merge gates
- Hallucinated architecture or API contracts: solve with typed contracts and validation from the architect agent
- False confidence from generated code: solve with executable validation, not prose
- Over-automation too early: keep human approval in the loop
