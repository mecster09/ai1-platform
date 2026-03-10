# 08 Governance And Operations

## Goal

Add policy enforcement, observability, replay support, and cleanup so the platform is dependable.

## Target Stack

- Services: `Node.js` + `TypeScript`
- Workflow visibility: `Temporal`
- State and metadata: `SQLite`
- Runtime evidence: filesystem artifacts and logs
- Operational scope: local-first single-node deployment

## Tasks

### T-047 Policy Checks

- Outcome: Validation goes beyond tests and enforces architectural guardrails.
- Dependencies: `tasks/03-workflows.md#t-026-validate-delivery-workflow`, `tasks/06-delivery-agents.md#t-040-front-end-agent-worker`, `tasks/06-delivery-agents.md#t-041-back-end-agent-worker`

- [ ] Implement unapproved API usage checks
- [ ] Implement contract drift checks
- [ ] Implement file ownership boundary checks
- [ ] Implement unapproved schema change checks
- [ ] Integrate policy checks into validation workflow

### T-048 Observability

- Outcome: Story, run, and workflow state can be debugged end to end.
- Dependencies: `tasks/03-workflows.md#t-020-workflow-worker-bootstrap`, `tasks/02-platform-api-and-web.md#t-012-platform-api-skeleton`, `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/05-agent-runtime-and-core-agents.md#t-035-shared-agent-runtime`

- [ ] Add structured logging across services
- [ ] Add correlation IDs for story, run, workflow, and task
- [ ] Add Temporal search attributes
- [ ] Add run-level telemetry and metrics
- [ ] Verify logs support end-to-end debugging

### T-049 Replay And Debugging

- Outcome: Failed workflows can be analyzed and re-driven without manual reconstruction.
- Dependencies: `tasks/03-workflows.md#t-028-parent-story-delivery-workflow`, `T-048`

- [ ] Add workflow inspection helpers
- [ ] Add artifact lookup helpers for failed runs
- [ ] Add blocked-run debugging support
- [ ] Verify replay-safe behavior for persisted activities

### T-050 Retention And Cleanup

- Outcome: The local platform remains usable over time without manual maintenance.
- Dependencies: `tasks/04-context-and-workspace.md#t-032-workspace-service`, `tasks/07-review-traceability-and-validation-ux.md#t-043-run-tracking`, `T-048`

- [ ] Add artifact retention rules
- [ ] Add workspace cleanup rules
- [ ] Add log cleanup rules
- [ ] Add context index cleanup rules
- [ ] Add scheduled cleanup jobs or scripts
