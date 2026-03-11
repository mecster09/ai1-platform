# 07 Delivery Agents

## Goal

Build the current built-in delivery agents with strict contracts and workspace integration.

## Target Stack

- Front-end agent target: `Next.js` + `TypeScript`
- Back-end agent target: `Node.js` + `TypeScript`
- Test automation: `Playwright`
- Worker runtime: `Node.js` + `TypeScript`
- Contract source: [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Tasks

### T-043 Front-End Agent Worker

- Outcome: The platform can generate traceable Next.js code changes from approved architecture.
- Dependencies: `tasks/06-agent-runtime-and-core-agents.md#t-038-shared-agent-runtime`, `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/06-agent-runtime-and-core-agents.md#t-039-shared-tool-adapters`, `tasks/02-agent-registry.md#t-014a-agent-registry-runtime-integration`

- [ ] Scaffold `agents/frontend-agent-worker`
- [ ] Enforce approved blueprint and contract requirements
- [ ] Verify run is permitted by selected and enabled agent types
- [ ] Restrict writes to approved front-end paths
- [ ] Generate patch artifacts
- [ ] Detect and emit contract mismatches
- [ ] Generate component or integration tests where relevant
- [ ] Enforce Next.js App Router defaults, Server Components by default, and limited Client Component usage
- [ ] Prevent ad hoc polling or mutation paths that bypass approved command or streaming contracts
- [ ] Add worker-level tests

### T-044 Back-End Agent Worker

- Outcome: The platform can generate traceable backend changes from approved architecture.
- Dependencies: `tasks/06-agent-runtime-and-core-agents.md#t-038-shared-agent-runtime`, `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/06-agent-runtime-and-core-agents.md#t-039-shared-tool-adapters`, `tasks/02-agent-registry.md#t-014a-agent-registry-runtime-integration`

- [ ] Scaffold `agents/backend-agent-worker`
- [ ] Enforce approved blueprint and contract requirements
- [ ] Verify run is permitted by selected and enabled agent types
- [ ] Restrict schema changes to approved cases
- [ ] Generate patch artifacts
- [ ] Generate tests and stubs where relevant
- [ ] Update API contract artifacts where required
- [ ] Include idempotency, review concurrency, and artifact metadata requirements in updated API artifacts when relevant
- [ ] Add worker-level tests

### T-045 Test-Automation Agent Worker

- Outcome: Acceptance criteria can be converted into executable end-to-end coverage.
- Dependencies: `tasks/06-agent-runtime-and-core-agents.md#t-038-shared-agent-runtime`, `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/06-agent-runtime-and-core-agents.md#t-039-shared-tool-adapters`, `tasks/02-agent-registry.md#t-014a-agent-registry-runtime-integration`

- [ ] Scaffold `agents/test-agent-worker`
- [ ] Enforce acceptance-criteria and blueprint input requirements
- [ ] Verify run is permitted by selected and enabled agent types
- [ ] Generate `TestScenario[]`
- [ ] Generate Playwright specs, fixtures, and helpers
- [ ] Generate acceptance-criteria coverage mapping
- [ ] Generate evidence artifacts and explicit links from acceptance criteria to test evidence
- [ ] Define trace, screenshot, and report artifact policy for generated test runs
- [ ] Record blockers and environment assumptions
- [ ] Add worker-level tests
