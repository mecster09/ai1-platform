# 08 Review Traceability And Validation UX

## Goal

Make runs, approvals, artifacts, validation, and traceability visible and actionable in the product.

## Target Stack

- Front end: `Next.js` + `TypeScript`
- API: `Node.js` + `TypeScript`
- Review and traceability state: `SQLite`
- Artifact and evidence source: filesystem + workflow outputs
- UX input sources: workflows, runs, reviews, artifacts, validation reports

## Tasks

### T-046 Run Tracking

- Outcome: Users can inspect the state and output of each run.
- Dependencies: `tasks/03-platform-api-and-web.md#t-016-public-api-surface`, `tasks/03-platform-api-and-web.md#t-021-story-detail-and-status-ui`, `tasks/04-workflows.md#t-024-shared-activities`

- [ ] Add API support for run detail views
- [ ] Add API support for run logs
- [ ] Add API support for run event and log streaming
- [ ] Add API support for changed files and artifacts
- [ ] Build UI for run summaries and timelines
- [ ] Build UI for run logs and outputs
- [ ] Use live streaming for operational views with polling only as fallback

### T-047 Approval UX

- Outcome: Review decisions are manageable in the product.
- Dependencies: `tasks/06-agent-runtime-and-core-agents.md#t-042-approval-integration`, `tasks/03-platform-api-and-web.md#t-021-story-detail-and-status-ui`

- [ ] Build clarification review UI
- [ ] Build architecture approval UI
- [ ] Build revision request UI
- [ ] Build rejection UX
- [ ] Build final review UI
- [ ] Handle optimistic concurrency failures when review versions are stale

### T-048 Traceability

- Outcome: The traceability matrix promised by the plan is visible and queryable.
- Dependencies: `tasks/01-contracts-and-persistence.md#t-011-repository-modules`, `tasks/03-platform-api-and-web.md#t-021-story-detail-and-status-ui`, `tasks/04-workflows.md#t-026-decompose-story-workflow`

- [ ] Persist trace links between stories, criteria, tasks, runs, artifacts, and tests
- [ ] Persist selected agent set links for story and run views
- [ ] Add API support for traceability queries
- [ ] Build traceability matrix UI
- [ ] Verify backward and forward traceability paths

### T-049 Diff And Artifact Review

- Outcome: Final review has the evidence needed for approval.
- Dependencies: `tasks/03-platform-api-and-web.md#t-018-artifact-storage`, `T-046`, `T-047`

- [ ] Build diff viewing UI
- [ ] Build artifact browsing UI
- [ ] Build validation evidence views
- [ ] Build review package assembly screens
- [ ] Distinguish artifact metadata views from authorized artifact download flows
- [ ] Verify final review has all required evidence
