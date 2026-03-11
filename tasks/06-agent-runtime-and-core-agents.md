# 06 Agent Runtime And Core Agents

## Goal

Build the shared agent runtime plus the first two core agents: tasks and architect.

## Target Stack

- Agent workers: `Node.js` + `TypeScript`
- Agent runtime: `LangGraph` or a thin custom runner
- Context source: `context-service` + `ChromaDB`
- Policy and output source: [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Tasks

### T-038 Shared Agent Runtime

- Outcome: All agents share the same enforcement model.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`, `tasks/01-contracts-and-persistence.md#t-007-json-schema-package`, `tasks/01-contracts-and-persistence.md#t-008-runtime-validation-helpers`, `tasks/02-agent-registry.md#t-012-agent-registry-contracts`

- [ ] Create `@ai1/agent-runtime`
- [ ] Parse and validate shared run envelopes
- [ ] Parse and validate `availableAgents`
- [ ] Enforce policy constraints
- [ ] Enforce allowed tool use
- [ ] Enforce agent-type checks against configured registry data
- [ ] Validate agent outputs against schemas
- [ ] Standardize blocked and failed responses
- [ ] Decide per agent whether to use `LangGraph` or a thin custom runner
- [ ] Ensure `LangGraph`, if used, remains agent-local and does not own story-level orchestration, approval flow, or retries
- [ ] Define or remove `apiBoundaryVersion` and `emittedEvents` as canonical runtime fields before enforcing them
- [ ] Define or remove `mustUseApprovedApiBoundary`, `requiresIdempotentMutations`, and `requiresReviewVersionCheck` before enforcing them

### T-039 Shared Tool Adapters

- Outcome: Agents can work through consistent controlled tools.
- Dependencies: `T-038`, `tasks/05-context-and-workspace.md#t-037-safe-command-execution`

- [ ] Add file read tools
- [ ] Add code search tools
- [ ] Add controlled write tools
- [ ] Add contract read tools
- [ ] Add diff inspection tools
- [ ] Add targeted test execution tools

### T-040 Tasks Agent Worker

- Outcome: The tasks agent can generate structured task graphs and ambiguity flags.
- Dependencies: `T-038`, `tasks/05-context-and-workspace.md#t-033-context-service`, `tasks/02-agent-registry.md#t-014a-agent-registry-runtime-integration`

- [ ] Scaffold `agents/tasks-agent-worker`
- [ ] Enforce strict tasks-agent input contract
- [ ] Implement story decomposition logic
- [ ] Read configured available agents from runtime context
- [ ] Emit `Task[]`, recommended agent types, dependencies, risk flags, and ambiguity flags
- [ ] Persist and publish decomposition artifacts
- [ ] Add worker-level tests

### T-041 Architect Agent Worker

- Outcome: The platform can produce machine-readable technical blueprints.
- Dependencies: `T-038`, `tasks/05-context-and-workspace.md#t-033-context-service`

- [ ] Scaffold `agents/architect-agent-worker`
- [ ] Enforce strict architect-agent input contract
- [ ] Generate `ArchitectureBlueprint`
- [ ] Generate `ApiContract`
- [ ] Generate `DataModelChange[]`
- [ ] Generate sequence flow and impact analysis artifacts
- [ ] Flag approval-required changes
- [ ] Classify API changes as command, query, streaming, or artifact boundary changes where applicable
- [ ] Add worker-level tests

### T-042 Approval Integration

- Outcome: Human approval is part of the system, not an informal side channel.
- Dependencies: `tasks/03-platform-api-and-web.md#t-016-public-api-surface`, `tasks/03-platform-api-and-web.md#t-019-platform-web-skeleton`, `tasks/04-workflows.md#t-023-workflow-worker-bootstrap`

- [ ] Connect review creation to `platform-api`
- [ ] Connect review states to `platform-web`
- [ ] Wire review decisions back to Temporal signals
- [ ] Use workflow signals for approval input and query or read-model paths for status inspection
- [ ] Support clarification reviews
- [ ] Support architecture approvals
- [ ] Support final review decisions
- [ ] Validate idempotency keys and review version checks for approval and revision flows
