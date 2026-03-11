# Server-Sent Events (SSE) Validation

## Scope

This document validates the AI1 Platform design assumptions about `Server-Sent Events (SSE)` against primary web platform documentation surfaced through Context7.

Context7 match:

- Library ID: `/mdn/content`

Validated on:

- March 11, 2026

Source basis:

- MDN documentation retrieved through Context7 for `EventSource`, SSE stream format, named events, reconnect behavior, and `text/event-stream`

## Validation Summary

The current documentation in `/docs` is well aligned with `Server-Sent Events (SSE)` as the preferred MVP transport for one-way operational updates from the platform to the browser.

The following assumptions are validated by the official docs:

- SSE is appropriate for one-way server-to-browser streaming.
- The browser client uses `EventSource`.
- The server sends a long-lived `text/event-stream` HTTP response.
- SSE is simpler than WebSockets when the browser only needs to receive updates.
- Browsers automatically attempt to reconnect unless the stream is explicitly closed.
- SSE supports named events, message IDs, and retry hints.

## What the Current Docs Get Right

### 1. SSE is a good fit for live operational updates

The platform docs propose SSE for:

- live run status
- workflow events
- log tails
- approval state changes

This matches SSE well because these are primarily server-to-browser update streams.

### 2. SSE is an appropriate MVP choice over WebSockets

The docs prefer SSE for MVP. That is a sound choice when the transport requirement is mostly:

- simple
- one-way
- browser-friendly
- low ceremony

This aligns with web platform guidance around `EventSource`.

### 3. Route/HTTP boundary usage is correct

Your broader `Next.js` architecture already places SSE under explicit HTTP boundaries. That is correct and consistent with how SSE works.

## Corrections and Tightening Needed

### 1. SSE is unidirectional only

This is the main architectural constraint.

SSE is excellent for server-to-client delivery, but it is not a bidirectional messaging protocol.

Recommendation:

- keep all client-to-server mutations on normal command paths such as:
  - Server Actions
  - command API endpoints
- use SSE only for observation and updates flowing back to the UI

### 2. Event naming should be standardized

The platform wants to stream different operational concerns through one transport.

SSE supports named events, which makes the stream easier to consume predictably.

Recommendation:

- define stable event types such as:
  - `run.status`
  - `run.log`
  - `approval.changed`
  - `validation.progress`
  - `workflow.completed`

### 3. Reconnect and resume behavior should be part of the contract

SSE supports reconnect behavior, `id` fields, and retry hints.

Recommendation:

- include event IDs in streamed messages
- define resume semantics for reconnecting clients
- define whether clients must backfill missed events from a query endpoint after reconnect

### 4. Keep-alive behavior should be planned

SSE streams can time out through intermediaries or idle periods.

Recommendation:

- send periodic comment or heartbeat lines
- define idle timeout and reconnect expectations explicitly

### 5. Log streaming format should be deliberate

SSE supports simple text and JSON payloads, but log tails can become noisy and unstructured.

Recommendation:

- stream structured log events as JSON payloads
- keep raw line-oriented logs available separately as artifacts if needed

## Implementation Guidance

Based on the validation, `SSE` is a strong fit if implementation follows these rules:

1. Use SSE only for server-to-browser streaming.
2. Keep browser-originated commands on normal API or action paths.
3. Serve streams as `text/event-stream` over explicit HTTP endpoints.
4. Use named events for distinct operational concerns.
5. Include event IDs and define reconnect behavior.
6. Send heartbeat comments to keep long-lived streams healthy.
7. Treat SSE as the live-view transport, not the source of truth for historical state.

## Result

Validation result: `pass`

`Server-Sent Events (SSE)` is a strong and appropriate match for the AI1 Platform’s MVP live-update requirements described in `/docs`. The current design is sound, with the main refinements being event taxonomy, reconnect semantics, and maintaining a strict one-way transport boundary.
