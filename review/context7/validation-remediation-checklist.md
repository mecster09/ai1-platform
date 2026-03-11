# Validation Remediation Checklist

## Purpose

This document consolidates the remediation items identified during stack validation across the documentation in `/docs`.

It focuses only on technologies that were assessed as:

- `pass with refinements`
- `pass with significant refinements`
- `pass with caution`

## Priority Order

Recommended remediation order:

1. `Temporal`
2. `LangGraph`
3. `Next.js`
4. `TypeScript`
5. `Node.js`
6. `SQLite`
7. `ChromaDB`

This order reflects architectural dependency and risk. The `Temporal` and `LangGraph` boundary should be resolved before implementation detail is allowed to spread into service, agent, and API design.

## Temporal

Source:

- [validation-temporal.md](/c:/Code/AI1-Platform/docs/validation-temporal.md)

Required refinements:

- State explicitly that `Temporal` workflows orchestrate business process only.
- State explicitly that tool execution, file mutation, network calls, and repository operations must occur in Activities or worker-side execution code, not Workflow code.
- Decide and document whether agent workers are:
  - Temporal Activity workers directly
  - external execution services invoked by Activities
- Standardize approval handling for MVP:
  - Signals for approval/rejection input
  - Queries or read models for status inspection
- Define workflow identity and idempotency rules:
  - workflow ID format
  - workflow ID reuse policy
  - mapping between API idempotency keys, story IDs, run IDs, and workflow starts

## LangGraph

Source:

- [validation-langgraph.md](/c:/Code/AI1-Platform/docs/validation-langgraph.md)

Required refinements:

- Define the boundary between `Temporal` and `LangGraph` explicitly.
- Keep `Temporal` as the outer business-process orchestrator.
- Use `LangGraph` only inside agent workers where graph-based agent execution is actually needed.
- Decide whether LangGraph checkpointing/durability is necessary for MVP or whether `Temporal` durability is sufficient at the outer layer.
- Ensure LangGraph tasks map to platform-controlled tools rather than bypassing file, approval, or execution policy.
- Keep the thin custom runner option open for simpler agents.

## Next.js

Source:

- [validation-nextjs.md](/c:/Code/AI1-Platform/docs/validation-nextjs.md)

Required refinements:

- Clarify that Server Actions do not bypass platform validation, authorization, audit, and idempotency rules.
- Clarify that Route Handlers are for explicit HTTP surfaces, not the default internal data access layer.
- Define per-screen rendering and freshness rules for operational views:
  - fully dynamic
  - no-store
  - bounded revalidation
- Keep live streaming concerns under explicit HTTP boundaries.
- Document how `platform-web` preserves the same invariants whether it uses HTTP or a shared typed server-side client.

## TypeScript

Source:

- [validation-typescript.md](/c:/Code/AI1-Platform/docs/validation-typescript.md)

Required refinements:

- State explicitly that TypeScript types are compile-time only.
- Do not treat TypeScript interfaces as sufficient runtime validation.
- Define which contract artifact is canonical at each boundary:
  - TypeScript types
  - `OpenAPI`
  - `JSON Schema`
- Define a shared-types strategy:
  - where shared domain contracts live
  - what is internal vs public
  - how versioning is managed
- Use type-only imports/exports where appropriate to reduce runtime coupling.

## Node.js

Source:

- [validation-nodejs.md](/c:/Code/AI1-Platform/docs/validation-nodejs.md)

Required refinements:

- Define structured command execution rules:
  - prefer `spawn` or `execFile`
  - avoid defaulting to shell-based `exec`
- Define execution controls:
  - timeout
  - working directory
  - environment propagation
  - output capture
  - exit code handling
- Clarify the service concurrency model:
  - separate process
  - async service
  - worker threads only where justified
- Constrain command and filesystem access to workspace/service boundaries.

## SQLite

Source:

- [validation-sqlite.md](/c:/Code/AI1-Platform/docs/validation-sqlite.md)

Required refinements:

- Make `WAL` mode an explicit default.
- Make `foreign_keys = ON` an explicit default.
- Define migration, backup, and integrity-check procedures.
- Keep concurrency expectations scoped to single-node local operation.
- Keep traceability-critical entities relational-first.
- Use JSON-in-column selectively rather than as a substitute for schema design.

## ChromaDB

Source:

- [validation-chromadb.md](/c:/Code/AI1-Platform/docs/validation-chromadb.md)

Required refinements:

- Reconfirm the design against current primary Chroma documentation before implementation lock-in.
- Keep `ChromaDB` explicitly non-authoritative.
- Define collection strategy.
- Define retrieval metadata dimensions, such as:
  - `storyId`
  - `runId`
  - `agentType`
  - `artifactType`
  - `repo`
  - `module`
  - `documentKind`
  - `schemaVersion`
- Define indexing lifecycle:
  - chunking
  - embedding model selection
  - re-index triggers
  - stale artifact cleanup
- Record retrieval provenance per run.

## Cross-Cutting Actions

These items cut across multiple validations and should be handled centrally:

- Define authoritative contract boundaries across `Temporal`, `LangGraph`, `TypeScript`, `OpenAPI`, and `JSON Schema`.
- Define a single policy model for:
  - approval
  - file mutation
  - external command execution
  - schema changes
  - API changes
- Define idempotency and identity semantics consistently across:
  - API commands
  - workflow executions
  - agent runs
  - streamed events
  - artifact records
- Define observability and evidence conventions consistently across:
  - logs
  - validation results
  - Playwright artifacts
  - workflow visibility
  - streamed operational events

## Suggested Next Step

Update the core architecture documents to absorb these remediations in this order:

1. [architecture.md](/c:/Code/AI1-Platform/docs/architecture.md)
2. [plan.md](/c:/Code/AI1-Platform/docs/plan.md)
3. [tech-stack.md](/c:/Code/AI1-Platform/docs/tech-stack.md)

After that, re-run validation only for the documents and technologies affected by those edits.
