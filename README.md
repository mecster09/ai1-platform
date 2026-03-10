# AI1 Platform

AI1 Platform is a local-first software delivery platform for turning structured stories into planned, implemented, validated, and reviewable software changes.

At a high level, the platform combines:

- a shared workflow engine
- a shared project memory and context layer
- a shared repo and workspace model
- a human approval layer
- preconfigured specialist agents for decomposition, architecture, implementation, and test automation

The platform is designed around three layers:

- Platform layer: the local control plane for workflows, persistence, context retrieval, artifacts, approvals, and UI
- Delivery intelligence layer: the shared domain model, contracts, and traceability structure used by all agents
- Specialist agent layer: configured agents made available through a platform-managed registry, with tasks and architecture as control agents and delivery agents selected per story

The intended local-first stack is:

- `Temporal` for orchestration, retries, and visibility
- `LangGraph` or a thin custom runner for agent execution
- `SQLite` for structured metadata and workflow-linked state
- `ChromaDB` for local vector retrieval
- filesystem-based artifact storage
- `Next.js` for the platform UI

The operating model is workflow-and-contract based rather than free-form agent chat:

1. A story is submitted.
2. The workflow loads the configured agent registry.
3. The tasks agent decomposes the story into structured work and recommends delivery agents from the configured set.
4. The architect agent produces the blueprint and contracts.
5. Only the selected delivery agents execute where dependencies allow.
6. Validation and review gates run.
7. A human approves, rejects, or requests revision.

The current built-in delivery-agent set is:

- `frontend`
- `backend`
- `test-automation`

The platform is intended to support `frontend` only, `backend` only, `test-automation` only, any pair of those agents, or all three, with future agent types added through the same registry model.

Detailed planning and architecture documents live in [docs/plan.md](/c:/Code/AI1-Platform/docs/plan.md) and [docs/architecture.md](/c:/Code/AI1-Platform/docs/architecture.md).
