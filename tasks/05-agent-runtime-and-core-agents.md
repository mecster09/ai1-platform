# 05 Agent Runtime And Core Agents

## Goal

Build the shared agent runtime plus the first two core agents: tasks and architect.

## Target Stack

- Agent workers: `Node.js` + `TypeScript`
- Agent runtime: `LangGraph` or a thin custom runner
- Context source: `context-service` + `ChromaDB`
- Policy and output source: [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Tasks

### T-035 Shared Agent Runtime

- Outcome: All agents share the same enforcement model.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`, `tasks/01-contracts-and-persistence.md#t-007-json-schema-package`, `tasks/01-contracts-and-persistence.md#t-008-runtime-validation-helpers`

- [ ] Create `@ai1/agent-runtime`
- [ ] Parse and validate shared run envelopes
- [ ] Enforce policy constraints
- [ ] Enforce allowed tool use
- [ ] Validate agent outputs against schemas
- [ ] Standardize blocked and failed responses

### T-036 Shared Tool Adapters

- Outcome: Agents can work through consistent controlled tools.
- Dependencies: `T-035`, `tasks/04-context-and-workspace.md#t-034-safe-command-execution`

- [ ] Add file read tools
- [ ] Add code search tools
- [ ] Add controlled write tools
- [ ] Add contract read tools
- [ ] Add diff inspection tools
- [ ] Add targeted test execution tools

### T-037 Tasks Agent Worker

- Outcome: The tasks agent can generate structured task graphs and ambiguity flags.
- Dependencies: `T-035`, `tasks/04-context-and-workspace.md#t-030-context-service`

- [ ] Scaffold `agents/tasks-agent-worker`
- [ ] Enforce strict tasks-agent input contract
- [ ] Implement story decomposition logic
- [ ] Emit `Task[]`, dependencies, risk flags, and ambiguity flags
- [ ] Persist and publish decomposition artifacts
- [ ] Add worker-level tests

### T-038 Architect Agent Worker

- Outcome: The platform can produce machine-readable technical blueprints.
- Dependencies: `T-035`, `tasks/04-context-and-workspace.md#t-030-context-service`

- [ ] Scaffold `agents/architect-agent-worker`
- [ ] Enforce strict architect-agent input contract
- [ ] Generate `ArchitectureBlueprint`
- [ ] Generate `ApiContract`
- [ ] Generate `DataModelChange[]`
- [ ] Generate sequence flow and impact analysis artifacts
- [ ] Flag approval-required changes
- [ ] Add worker-level tests

### T-039 Approval Integration

- Outcome: Human approval is part of the system, not an informal side channel.
- Dependencies: `tasks/02-platform-api-and-web.md#t-013-public-api-surface`, `tasks/02-platform-api-and-web.md#t-016-platform-web-skeleton`, `tasks/03-workflows.md#t-020-workflow-worker-bootstrap`

- [ ] Connect review creation to `platform-api`
- [ ] Connect review states to `platform-web`
- [ ] Wire review decisions back to Temporal signals
- [ ] Support clarification reviews
- [ ] Support architecture approvals
- [ ] Support final review decisions
