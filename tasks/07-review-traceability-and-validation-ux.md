# 07 Review Traceability And Validation UX

## Goal

Make runs, approvals, artifacts, validation, and traceability visible and actionable in the product.

## Target Stack

- Front end: `Next.js` + `TypeScript`
- API: `Node.js` + `TypeScript`
- Review and traceability state: `SQLite`
- Artifact and evidence source: filesystem + workflow outputs
- UX input sources: workflows, runs, reviews, artifacts, validation reports

## Tasks

### T-043 Run Tracking

- Outcome: Users can inspect the state and output of each run.
- Dependencies: `tasks/02-platform-api-and-web.md#t-013-public-api-surface`, `tasks/02-platform-api-and-web.md#t-018-story-detail-and-status-ui`, `tasks/03-workflows.md#t-021-shared-activities`

- [ ] Add API support for run detail views
- [ ] Add API support for run logs
- [ ] Add API support for changed files and artifacts
- [ ] Build UI for run summaries and timelines
- [ ] Build UI for run logs and outputs

### T-044 Approval UX

- Outcome: Review decisions are manageable in the product.
- Dependencies: `tasks/05-agent-runtime-and-core-agents.md#t-039-approval-integration`, `tasks/02-platform-api-and-web.md#t-018-story-detail-and-status-ui`

- [ ] Build clarification review UI
- [ ] Build architecture approval UI
- [ ] Build revision request UI
- [ ] Build rejection UX
- [ ] Build final review UI

### T-045 Traceability

- Outcome: The traceability matrix promised by the plan is visible and queryable.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-011-repository-modules`, `tasks/02-platform-api-and-web.md#t-018-story-detail-and-status-ui`, `tasks/06-delivery-agents.md#t-042-test-automation-agent-worker`

- [ ] Persist trace links between stories, criteria, tasks, runs, artifacts, and tests
- [ ] Add API support for traceability queries
- [ ] Build traceability matrix UI
- [ ] Verify backward and forward traceability paths

### T-046 Diff And Artifact Review

- Outcome: Final review has the evidence needed for approval.
- Dependencies: `tasks/02-platform-api-and-web.md#t-015-artifact-storage`, `T-043`, `T-044`

- [ ] Build diff viewing UI
- [ ] Build artifact browsing UI
- [ ] Build validation evidence views
- [ ] Build review package assembly screens
- [ ] Verify final review has all required evidence
