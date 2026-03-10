# 06 Delivery Agents

## Goal

Build the front-end, back-end, and test-automation agents with strict contracts and workspace integration.

## Target Stack

- Front-end agent target: `Next.js` + `TypeScript`
- Back-end agent target: `Node.js` + `TypeScript`
- Test automation: `Playwright`
- Worker runtime: `Node.js` + `TypeScript`
- Contract source: [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Tasks

### T-040 Front-End Agent Worker

- Outcome: The platform can generate traceable Next.js code changes from approved architecture.
- Dependencies: `tasks/05-agent-runtime-and-core-agents.md#t-035-shared-agent-runtime`, `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/05-agent-runtime-and-core-agents.md#t-036-shared-tool-adapters`

- [ ] Scaffold `agents/frontend-agent-worker`
- [ ] Enforce approved blueprint and contract requirements
- [ ] Restrict writes to approved front-end paths
- [ ] Generate patch artifacts
- [ ] Detect and emit contract mismatches
- [ ] Generate component or integration tests where relevant
- [ ] Add worker-level tests

### T-041 Back-End Agent Worker

- Outcome: The platform can generate traceable backend changes from approved architecture.
- Dependencies: `tasks/05-agent-runtime-and-core-agents.md#t-035-shared-agent-runtime`, `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/05-agent-runtime-and-core-agents.md#t-036-shared-tool-adapters`

- [ ] Scaffold `agents/backend-agent-worker`
- [ ] Enforce approved blueprint and contract requirements
- [ ] Restrict schema changes to approved cases
- [ ] Generate patch artifacts
- [ ] Generate tests and stubs where relevant
- [ ] Update API contract artifacts where required
- [ ] Add worker-level tests

### T-042 Test-Automation Agent Worker

- Outcome: Acceptance criteria can be converted into executable end-to-end coverage.
- Dependencies: `tasks/05-agent-runtime-and-core-agents.md#t-035-shared-agent-runtime`, `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/05-agent-runtime-and-core-agents.md#t-036-shared-tool-adapters`

- [ ] Scaffold `agents/test-agent-worker`
- [ ] Enforce acceptance-criteria and blueprint input requirements
- [ ] Generate `TestScenario[]`
- [ ] Generate Playwright specs, fixtures, and helpers
- [ ] Generate acceptance-criteria coverage mapping
- [ ] Record blockers and environment assumptions
- [ ] Add worker-level tests
