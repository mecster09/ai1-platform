# 02 Agent Registry

## Goal

Implement the configured agent registry that makes enabled agents available to workflows and agent workers.

## Target Stack

- Language: `TypeScript`
- Metadata store: `SQLite`
- API and workflow consumers: `Node.js` + `TypeScript`
- Contract sources: [plan.md](c:/Code/AI1-Platform/docs/plan.md), [architecture.md](c:/Code/AI1-Platform/docs/architecture.md), [agent-contracts.md](c:/Code/AI1-Platform/docs/agent-contracts.md)

## Tasks

### T-012 Agent Registry Contracts

- Outcome: The platform has typed definitions for configured agents and registry-backed availability.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-006-typescript-domain-contracts`, `tasks/01-contracts-and-persistence.md#t-007-json-schema-package`

- [ ] Add contract types for configured agent definitions
- [ ] Add schema definitions for registry-backed agent availability
- [ ] Add `availableAgents` support to agent run envelope types
- [ ] Add `recommendedAgentTypes` and `selectedAgentTypes` support where required
- [ ] Add contract-level tests for registry-backed agent data

### T-013 Agent Registry Persistence And Loading

- Outcome: Configured agents can be stored, loaded, and provided to workflows and workers.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-009-sqlite-access-layer`, `tasks/01-contracts-and-persistence.md#t-010-initial-sqlite-schema`, `T-012`

- [ ] Add persistence approach for configured agent registry data
- [ ] Add repository access for configured agents
- [ ] Add startup-time loading of configured agents
- [ ] Add filtering for enabled versus disabled agents
- [ ] Verify configured registry data is available to platform services

### T-014A Agent Registry Runtime Integration

- Outcome: Workflow workers and agent runs receive the configured registry or an appropriate filtered view without creating a bootstrap cycle.
- Dependencies: `T-013`

- [ ] Add workflow input support for configured agent registry references
- [ ] Add workflow-worker integration for passing registry data to activities
- [ ] Add agent-run envelope population for `availableAgents`
- [ ] Verify workflows and agents only see configured available agents

### T-014B Agent Registry API Integration

- Outcome: The public API can expose and use configured registry data after the runtime path exists.
- Dependencies: `T-014A`, `tasks/03-platform-api-and-web.md#t-015-platform-api-skeleton`

- [ ] Add platform API support for loading configured agent registry data
- [ ] Support registry-backed selection and visibility in API responses where required
