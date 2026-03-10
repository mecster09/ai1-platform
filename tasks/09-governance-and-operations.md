# 09 Governance And Operations

## Goal

Add policy enforcement, observability, replay support, and cleanup so the platform is dependable.

## Target Stack

- Services: `Node.js` + `TypeScript`
- Workflow visibility: `Temporal`
- State and metadata: `SQLite`
- Runtime evidence: filesystem artifacts and logs
- Operational scope: local-first single-node deployment

## Tasks

### T-050 Policy Checks

- Outcome: Validation goes beyond tests and enforces architectural guardrails.
- Dependencies: `tasks/04-workflows.md#t-029-validate-delivery-workflow`, `tasks/07-delivery-agents.md#t-043-front-end-agent-worker`, `tasks/07-delivery-agents.md#t-044-back-end-agent-worker`, `tasks/02-agent-registry.md#t-013-agent-registry-persistence-and-loading`

- [ ] Implement unapproved API usage checks
- [ ] Implement contract drift checks
- [ ] Implement file ownership boundary checks
- [ ] Implement unapproved schema change checks
- [ ] Implement dispatch checks against configured and enabled agent types
- [ ] Implement checks for non-idempotent write routes and undocumented mutation semantics
- [ ] Implement checks for undocumented streaming endpoints or ad hoc polling paths
- [ ] Implement checks for missing review-version enforcement on approval commands
- [ ] Integrate policy checks into validation workflow

### T-051 Observability

- Outcome: Story, run, and workflow state can be debugged end to end.
- Dependencies: `tasks/04-workflows.md#t-023-workflow-worker-bootstrap`, `tasks/03-platform-api-and-web.md#t-015-platform-api-skeleton`, `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/06-agent-runtime-and-core-agents.md#t-038-shared-agent-runtime`

- [ ] Add structured logging across services
- [ ] Add correlation IDs for story, run, workflow, and task
- [ ] Add Temporal search attributes
- [ ] Add run-level telemetry and metrics
- [ ] Add telemetry for streaming connections, published run events, and review command failures
- [ ] Add Next.js framework instrumentation via `instrumentation.ts`
- [ ] Verify logs support end-to-end debugging

### T-052 Replay And Debugging

- Outcome: Failed workflows can be analyzed and re-driven without manual reconstruction.
- Dependencies: `tasks/04-workflows.md#t-031-parent-story-delivery-workflow`, `T-051`

- [ ] Add workflow inspection helpers
- [ ] Add artifact lookup helpers for failed runs
- [ ] Add blocked-run debugging support
- [ ] Verify replay-safe behavior for persisted activities

### T-053 Retention And Cleanup

- Outcome: The local platform remains usable over time without manual maintenance.
- Dependencies: `tasks/05-context-and-workspace.md#t-035-workspace-service`, `tasks/08-review-traceability-and-validation-ux.md#t-046-run-tracking`, `T-051`

- [ ] Add artifact retention rules
- [ ] Add workspace cleanup rules
- [ ] Add log cleanup rules
- [ ] Add context index cleanup rules
- [ ] Add scheduled cleanup jobs or scripts
