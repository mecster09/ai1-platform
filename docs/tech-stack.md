# AI1 Platform Tech Stack

## Purpose

This document consolidates the technologies referenced across the platform planning and architecture documents into a single implementation-oriented stack reference.

It distinguishes between:

- Core adopted technologies
- Supporting standards and platform features
- Explicitly deferred technologies

## Core Stack

### Application Frameworks

- UI framework: `Next.js`
- UI library: `React`
- Primary language: `TypeScript`
- Platform service runtime: `Node.js`

### Workflow and Agent Runtime

- Workflow orchestration: `Temporal`
- Agent execution runtime: `LangGraph` or a thin custom runner inside individual agent workers

Boundary rule:

- `Temporal` owns story-level workflow orchestration, retries, approval waits, and multi-agent coordination.
- `LangGraph`, if used, owns only agent-local reasoning and tool-routing flow.

### Model Providers and AI Access

- Provider support: `OpenAI`
- Provider support: `Anthropic`
- Compatibility target: OpenAI-compatible model endpoints

Recommended shared environment variables:

- `AI_PROVIDER`
- `AI_MODEL`
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `OPENAI_BASE_URL`

### Data and Storage

- Relational metadata store: `SQLite`
- Vector and retrieval store: `ChromaDB`
- Artifact storage: local filesystem or object-style local storage

Implementation defaults:

- `SQLite` should enable WAL mode.
- `SQLite` should enable foreign key enforcement.
- `ChromaDB` should be treated as retrieval infrastructure, not a source of truth.

### Source Control and Workspace Execution

- Version control: `Git`
- Package and script execution: `npm`
- Isolated execution: `Docker`-based workers or equivalent isolated local workers

Execution policy:

- External commands should use structured execution semantics where possible rather than shell-heavy invocation.
- Command execution should capture output, exit code, duration, working directory, and applied timeouts.

### Testing and Validation

- End-to-end testing: `Playwright`
- Additional validation: existing lint, typecheck, and unit test frameworks in the target repository

## Web Platform Conventions

For `Next.js`-based platform UI development, the architecture documents assume:

- App Router
- Server Components by default
- Client Components only where interactivity requires them
- Server Actions for same-origin mutations
- Route Handlers for explicit HTTP boundaries
- `instrumentation.ts` for framework-level telemetry integration
- `Server-Sent Events (SSE)` as the preferred MVP streaming transport

UI boundary rules:

- Server Actions must preserve the same validation, authorization, audit, and idempotency semantics as the platform API boundary.
- Route Handlers are for explicit HTTP surfaces, not the default data-access layer for server-rendered pages.

## Contract and Schema Standards

The platform documentation references the following contract formats and standards:

- `OpenAPI`
- `JSON Schema`
- typed `TypeScript` interfaces

These are intended to support stable contracts between platform services and agent outputs, especially across front-end, back-end, and validation workflows.

Contract rule:

- `TypeScript` types improve implementation safety but do not replace runtime validation.
- Canonical contract format should be defined per boundary rather than assumed globally.

## Recommended Service Set

The current documentation implies the following service and worker footprint:

- `platform-web`
- `platform-api`
- `workflow-worker`
- `tasks-agent-worker`
- `architect-agent-worker`
- `frontend-agent-worker`
- `backend-agent-worker`
- `test-agent-worker`
- `context-service`
- `workspace-service`

## Deferred or Avoided Early

The planning documents explicitly advise against introducing these too early:

- `Kafka`
- `Kubernetes`
- multiple overlapping agent frameworks
- free-form multi-agent chat architectures

## Selection Summary

The stack favors:

- local-first execution
- typed contracts over prompt-only coordination
- workflow orchestration over agent-to-agent conversation
- deterministic validation over self-reported completion
- conservative operational dependencies for MVP delivery
