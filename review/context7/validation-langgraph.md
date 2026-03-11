# LangGraph Validation

## Scope

This document validates the AI1 Platform design assumptions about `LangGraph` against the official LangGraph documentation surfaced through Context7.

Context7 match:

- Library ID: `/websites/langchain_oss_javascript_langgraph`

Validated on:

- March 11, 2026

Source basis:

- Official LangGraph JavaScript documentation retrieved through Context7 for state graphs, tasks, durable execution, interrupts, and human-in-the-loop behavior

## Validation Summary

The current documentation in `/docs` is directionally aligned with `LangGraph` as a controlled agent execution runtime, but this choice needs tighter boundary definition than the other technologies validated so far.

The following assumptions are validated by the official docs:

- `LangGraph` supports stateful graph-based execution.
- It supports tool-driven agent flows.
- It supports controllable branching and explicit execution graphs.
- It supports human-in-the-loop interruption and resumption.
- It supports persistence and resumability when used with a checkpointer.
- It expects determinism/idempotency discipline and wrapping side effects in tasks.

## What the Current Docs Get Right

### 1. LangGraph is a plausible fit for agent execution

The plan and tech stack describe `LangGraph` as:

- an agent execution runtime
- a mechanism for controlled agent execution
- an alternative to a thin custom runner

That matches the product well. LangGraph is explicitly designed for stateful, tool-using, interruptible agent flows.

### 2. Controlled execution over free-form agent chat

Your docs strongly prefer workflow-and-contract-driven execution over open-ended agent conversation. LangGraph aligns with that preference because it uses explicit graph structure and node transitions rather than unconstrained chat loops.

### 3. Human-in-the-loop agent control

The platform emphasizes approval gates and controlled pauses. LangGraph’s interrupt/resume model fits this style of agent runtime.

### 4. Side-effect discipline

The docs already emphasize determinism, contracts, and constrained tooling. LangGraph documentation explicitly reinforces that side effects should be wrapped in tasks and that resumable workflows should be deterministic and idempotent.

## Corrections and Tightening Needed

### 1. LangGraph and Temporal overlap significantly

This is the biggest architectural issue.

Your docs currently propose:

- `Temporal` for durable workflow orchestration
- `LangGraph` for agent execution runtime

That can work, but only if the boundary is explicit. LangGraph itself supports:

- durable execution
- interrupts
- resumability
- stateful orchestration

Without a clear split, you risk building two orchestration layers that both try to own:

- retries
- pause/resume behavior
- long-running state
- human approval flow

Recommendation:

- keep `Temporal` as the outer business-process orchestrator
- use `LangGraph` only inside an agent worker to model the internal reasoning/tool loop of that specific agent
- do not let LangGraph become a second platform-wide workflow engine

### 2. Choose whether LangGraph durability is required when Temporal already exists

Because `Temporal` already provides durable orchestration, you may not need the full durability story inside LangGraph for MVP.

Recommendation:

- decide early whether agent workers need:
  - simple in-process LangGraph execution
  - or checkpointed LangGraph execution for resumable agent internals

Do not adopt LangGraph durability features by default unless there is a clear benefit over the outer Temporal guarantees.

### 3. Define task/tool boundaries carefully

LangGraph docs emphasize wrapping side effects in tasks. Your platform also has strong constraints around file edits, repo mutation, and external tool execution.

Recommendation:

- map LangGraph tasks to platform-controlled tool abstractions
- do not let LangGraph nodes directly bypass platform policies for file mutation, test execution, or approvals

### 4. Keep the “thin custom runner” option alive until requirements are proven

Your docs already mention `LangGraph` or a thin custom runner. That is the right hedge.

Recommendation:

- do not commit to LangGraph everywhere before validating that each agent actually needs graph-level complexity
- simpler deterministic agents may be better served by a narrow custom runner

## Implementation Guidance

Based on the validation, `LangGraph` is a viable fit if implementation follows these rules:

1. Use `LangGraph` only as an inner agent runtime, not as the primary platform workflow engine.
2. Keep `Temporal` responsible for story-level orchestration, approvals, retries, and multi-agent coordination.
3. Use LangGraph for agent-local state transitions, tool routing, and optional human-in-the-loop agent pauses.
4. Wrap side effects in explicit tasks and keep those tasks policy-constrained.
5. Decide explicitly whether LangGraph checkpointing is necessary in MVP or whether Temporal-level durability is sufficient.
6. Retain the option to replace LangGraph with a thin custom runner for simpler agents.

## Result

Validation result: `pass with significant refinements`

`LangGraph` is a valid candidate for the AI1 Platform, but only with a sharply defined boundary relative to `Temporal`. The current docs are directionally correct in treating it as an agent runtime, yet they need stronger implementation guidance to prevent orchestration overlap and unnecessary architectural duplication.
