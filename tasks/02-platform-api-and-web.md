# 02 Platform API And Web

## Goal

Build the initial `platform-api` and `platform-web` so stories can be created, stored, and viewed.

## Target Stack

- Front end: `Next.js` + `TypeScript`
- API: `Node.js` + `TypeScript`
- Metadata store: `SQLite`
- Artifact storage: filesystem
- Contract source: [api-contract.json](c:/Code/AI1-Platform/docs/architecture/api-contract.json)

## Tasks

### T-012 Platform API Skeleton

- Outcome: A production-shaped API service skeleton exists.
- Dependencies: `tasks/00-foundation.md#t-002-repo-structure`, `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Scaffold `apps/platform-api` in Node.js + TypeScript
- [ ] Add config loading
- [ ] Add structured logging
- [ ] Add error handling and response shaping
- [ ] Add health endpoint

### T-013 Public API Surface

- Outcome: `platform-api` exposes the planned command and query boundary.
- Dependencies: `T-012`, `tasks/01-contracts-and-persistence.md#t-008-runtime-validation-helpers`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`

- [ ] Implement stories resource routes
- [ ] Implement artifact routes
- [ ] Implement workflow start routes
- [ ] Implement runs query routes
- [ ] Implement review routes
- [ ] Implement traceability routes
- [ ] Validate routes against [api-contract.json](c:/Code/AI1-Platform/docs/architecture/api-contract.json)

### T-014 Story Intake API

- Outcome: Stories can be created and queried through the API.
- Dependencies: `T-013`

- [ ] Implement `POST /stories`
- [ ] Implement `GET /stories/{storyId}`
- [ ] Implement `GET /stories`
- [ ] Persist stories and acceptance criteria
- [ ] Return schema-valid API responses

### T-015 Artifact Storage

- Outcome: The artifact model is usable by workflows and review UI.
- Dependencies: `T-013`, `tasks/01-contracts-and-persistence.md#t-011-repository-modules`

- [ ] Add filesystem artifact storage
- [ ] Persist artifact metadata in SQLite
- [ ] Support story attachment uploads
- [ ] Add artifact path conventions under `artifacts/`
- [ ] Add artifact retrieval helpers

### T-016 Platform Web Skeleton

- Outcome: A minimal web app can communicate with `platform-api`.
- Dependencies: `tasks/00-foundation.md#t-002-repo-structure`, `tasks/00-foundation.md#t-003-shared-typescript-and-tooling`

- [ ] Scaffold `apps/platform-web` in Next.js + TypeScript
- [ ] Add app shell and routing
- [ ] Add shared API client layer
- [ ] Add loading and error states
- [ ] Add root layout and navigation

### T-017 Story Intake UI

- Outcome: Users can create stories through the UI.
- Dependencies: `T-014`, `T-016`

- [ ] Build create-story form
- [ ] Add acceptance criteria entry UX
- [ ] Add constraints, priority, affected modules, and design reference inputs
- [ ] Submit story to `platform-api`
- [ ] Show success and validation errors cleanly

### T-018 Story Detail and Status UI

- Outcome: The web app becomes the read surface for platform progress.
- Dependencies: `T-016`, `T-014`, `T-015`

- [ ] Build story details page
- [ ] Show current story status
- [ ] Show decomposition and architecture summary placeholders
- [ ] Show artifacts and run references
- [ ] Prepare page structure for approvals and traceability
