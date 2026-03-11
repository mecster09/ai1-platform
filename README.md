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
- `LangGraph` or a thin custom runner for agent execution inside individual agent workers
- `SQLite` for structured metadata and workflow-linked state
- `ChromaDB` for local vector retrieval
- filesystem-based artifact storage
- `Next.js` for the platform UI

Runtime boundaries:

- `Temporal` is the outer business-process orchestrator for story delivery, approvals, retries, and multi-agent coordination.
- `LangGraph`, if used, is agent-local only and must not become a second platform-wide workflow engine.
- `SQLite` is the source of truth for structured state and should run with WAL mode and foreign key enforcement enabled.
- `ChromaDB` is retrieval infrastructure only; approved artifacts, relational state, and current source remain authoritative.

The agent runtime should support provider-backed LLM access, including:

- `OpenAI`
- `Anthropic`
- local or OpenAI-compatible endpoints

The operating model is workflow-and-contract based rather than free-form agent chat:

1. A story is submitted.
2. `DeliverStoryWorkflow` starts and loads the configured agent registry.
3. The tasks agent decomposes the story into structured work and recommends delivery agents from the configured set.
4. The architect agent produces the blueprint and contracts.
5. Only the selected delivery agents execute where dependencies allow.
6. Validation and review gates run.
7. A human approves, rejects, or requests revision.

UI and API boundary rules:

- `platform-web` uses Server Components by default and Client Components only where interactivity requires them.
- Server Actions may be used for same-origin mutations, but they must preserve the same validation, authorization, audit, and idempotency semantics as the platform API boundary.
- Live operational views use SSE for one-way server-to-browser streaming with ordered, reconnect-aware events.

The current built-in delivery-agent set is:

- `frontend`
- `backend`
- `test-automation`

The platform is intended to support `frontend` only, `backend` only, `test-automation` only, any pair of those agents, or all three, with future agent types added through the same registry model.

## LLM Configuration

Provider and model configuration should be supplied through environment variables loaded by the platform services and agent workers.

Recommended configuration shape:

- `AI_PROVIDER` with values such as `openai`, `anthropic`, or `openai-compatible`
- `AI_MODEL` for the default model name used by the agent runtime
- `OPENAI_API_KEY` when `AI_PROVIDER=openai`
- `ANTHROPIC_API_KEY` when `AI_PROVIDER=anthropic`
- `OPENAI_BASE_URL` when using a local or OpenAI-compatible endpoint

Recommended setup flow:

1. Copy the environment template created by the foundation tasks into a local `.env` file.
2. Add the API key for the selected provider.
3. Set `AI_PROVIDER` and `AI_MODEL`.
4. Start `platform-api`, `workflow-worker`, and the agent workers so they all receive the same provider configuration.

The intent is that provider choice and model selection are configuration, not code changes.

Detailed planning and architecture documents live in [docs/plan.md](/c:/Code/AI1-Platform/docs/plan.md) and [docs/architecture.md](/c:/Code/AI1-Platform/docs/architecture.md).
