# AI1 Platform Tasks

This folder breaks down the planning set in `/docs` into trackable checkbox-based task files.

The task files assume the current architecture decisions now reflected in `/docs`:

- `DeliverStoryWorkflow` is the public workflow entrypoint for story delivery
- `Temporal` owns story-level orchestration
- `LangGraph`, if used, is agent-local only
- `SQLite` is authoritative structured state
- `ChromaDB` is retrieval-only and metadata-filtered
- SSE is the preferred live-update transport for operational views

Recommended order:

- [00-foundation.md](c:/Code/AI1-Platform/tasks/00-foundation.md)
- [01-contracts-and-persistence.md](c:/Code/AI1-Platform/tasks/01-contracts-and-persistence.md)
- [02-agent-registry.md](c:/Code/AI1-Platform/tasks/02-agent-registry.md)
- [03-platform-api-and-web.md](c:/Code/AI1-Platform/tasks/03-platform-api-and-web.md)
- [04-workflows.md](c:/Code/AI1-Platform/tasks/04-workflows.md)
- [05-context-and-workspace.md](c:/Code/AI1-Platform/tasks/05-context-and-workspace.md)
- [06-agent-runtime-and-core-agents.md](c:/Code/AI1-Platform/tasks/06-agent-runtime-and-core-agents.md)
- [07-delivery-agents.md](c:/Code/AI1-Platform/tasks/07-delivery-agents.md)
- [08-review-traceability-and-validation-ux.md](c:/Code/AI1-Platform/tasks/08-review-traceability-and-validation-ux.md)
- [09-governance-and-operations.md](c:/Code/AI1-Platform/tasks/09-governance-and-operations.md)

Important sequencing notes:

- `02-agent-registry.md` is split so runtime integration happens before API integration:
  - `T-014A` runtime integration
  - `T-014B` API integration
- `04-workflows.md` assumes story intake starts `DeliverStoryWorkflow`, not `CreateStoryWorkflow` directly
- `05-context-and-workspace.md` and `06-agent-runtime-and-core-agents.md` should be treated as architectural constraint tracks, not just implementation tracks, because they lock execution safety and runtime boundaries

Milestone mapping:

- MVP 1: `00` through `04` plus core parts of `05` and `06`
- MVP 2: `07` plus validation completion in `04`
- MVP 3: `08`
- MVP 4: `09`

If task files are updated again, re-check them against:

- [plan.md](/c:/Code/AI1-Platform/docs/plan.md)
- [architecture.md](/c:/Code/AI1-Platform/docs/architecture.md)
- [temporal-workflows.md](/c:/Code/AI1-Platform/docs/temporal-workflows.md)
- [api-contract.json](/c:/Code/AI1-Platform/docs/architecture/api-contract.json)
- [data-model.json](/c:/Code/AI1-Platform/docs/architecture/data-model.json)
