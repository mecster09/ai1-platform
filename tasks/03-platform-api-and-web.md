# 03 Platform API And Web

## Goal

Build the initial `platform-api` and `platform-web` so stories can be created, stored, and viewed.

## Target Stack

- Front end: `Next.js` + `TypeScript`
- API: `Node.js` + `TypeScript`
- Metadata store: `SQLite`
- Artifact storage: filesystem
- Contract source: [api-contract.json](c:/Code/AI1-Platform/docs/architecture/api-contract.json)

## Tasks

### T-015 Platform API Skeleton

- Outcome: A production-shaped API service skeleton exists.
- Dependencies: `tasks/00-foundation.md#t-002-repo-structure`, `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Scaffold `apps/platform-api` in Node.js + TypeScript
- [ ] Add config loading
- [ ] Add structured logging
- [ ] Add error handling and response shaping
- [ ] Add health endpoint
- [ ] Add API versioning convention for public routes
- [ ] Add middleware or helpers for correlation IDs, idempotency extraction, and authorization context

### T-016 Public API Surface

- Outcome: `platform-api` exposes the planned command, query, streaming, and artifact boundary.
- Dependencies: `T-015`, `tasks/01-contracts-and-persistence.md#t-008-runtime-validation-helpers`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`, `tasks/02-agent-registry.md#t-014-agent-registry-api-and-runtime-integration`

- [ ] Implement stories resource routes
- [ ] Implement artifact routes
- [ ] Implement workflow start routes
- [ ] Implement runs query routes
- [ ] Implement run event and log streaming routes
- [ ] Implement review routes
- [ ] Implement traceability routes
- [ ] Split handlers internally into command, query, streaming, and artifact modules
- [ ] Support registry-backed delivery selection in workflow start routes
- [ ] Enforce idempotency keys on retriable write routes
- [ ] Enforce review version checks on approve and revision-request routes
- [ ] Validate routes against [api-contract.json](c:/Code/AI1-Platform/docs/architecture/api-contract.json)

### T-017 Story Intake API

- Outcome: Stories can be created and queried through the API.
- Dependencies: `T-016`

- [ ] Implement `POST /stories`
- [ ] Implement `GET /stories/{storyId}`
- [ ] Implement `GET /stories`
- [ ] Implement `GET /stories/{storyId}/dashboard`
- [ ] Persist stories and acceptance criteria
- [ ] Return story-level selected agent types where available
- [ ] Deduplicate retried `POST /stories` requests using idempotency keys
- [ ] Return schema-valid API responses

### T-018 Artifact Storage

- Outcome: The artifact model is usable by workflows and review UI.
- Dependencies: `T-016`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`

- [ ] Add filesystem artifact storage
- [ ] Persist artifact metadata in SQLite
- [ ] Support story attachment uploads
- [ ] Add artifact path conventions under `artifacts/`
- [ ] Add artifact retrieval helpers
- [ ] Implement artifact metadata lookup separate from authorized artifact download
- [ ] Persist content type, size, and integrity hash metadata
- [ ] Implement diff and evidence retrieval helpers aligned with artifact routes

### T-019 Platform Web Skeleton

- Outcome: A minimal web app can communicate with `platform-api`.
- Dependencies: `tasks/00-foundation.md#t-002-repo-structure`, `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Scaffold `apps/platform-web` in Next.js + TypeScript
- [ ] Add app shell and routing
- [ ] Add shared API client layer
- [ ] Add Server Action support for same-origin UI mutations
- [ ] Add SSE client support for live run and log views
- [ ] Add loading and error states
- [ ] Add root layout and navigation
- [ ] Add `instrumentation.ts`

### T-020 Story Intake UI

- Outcome: Users can create stories through the UI.
- Dependencies: `T-017`, `T-019`

- [ ] Build create-story form
- [ ] Add acceptance criteria entry UX
- [ ] Add constraints, priority, affected modules, and design reference inputs
- [ ] Submit story to `platform-api` through the approved command boundary
- [ ] Include idempotency keys on story-create submissions
- [ ] Show success and validation errors cleanly

### T-021 Story Detail and Status UI

- Outcome: The web app becomes the read surface for platform progress.
- Dependencies: `T-019`, `T-017`, `T-018`

- [ ] Build story details page
- [ ] Show current story status
- [ ] Show decomposition and architecture summary placeholders
- [ ] Show selected agent set and configured availability summary
- [ ] Show artifacts and run references
- [ ] Render dynamic operational views with Server Components by default and Client Components only where interactivity requires them
- [ ] Prepare live status and log sections to consume streaming endpoints
- [ ] Prepare page structure for approvals and traceability
