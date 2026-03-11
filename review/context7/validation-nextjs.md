# Next.js Validation

## Scope

This document validates the AI1 Platform design assumptions about `Next.js` against the official Next.js documentation surfaced through Context7.

Context7 match:

- Library ID: `/vercel/next.js/v16.1.6`

Validated on:

- March 11, 2026

Source basis:

- Official Next.js documentation retrieved through Context7 for App Router, Server Components, Client Components, Server Actions, Route Handlers, dynamic rendering, streaming behavior, and `instrumentation.ts`

## Validation Summary

The current documentation in `/docs` is broadly aligned with modern `Next.js` App Router guidance.

The following assumptions are validated by the official docs:

- App Router is the correct architecture model for this style of application.
- Pages and layouts are Server Components by default.
- Client Components should be used only where interactivity or browser APIs are required.
- Server Actions are a valid mechanism for same-origin mutations initiated by UI forms or controls.
- Route Handlers are the correct explicit HTTP boundary in App Router applications.
- Dynamic operational views should avoid static rendering assumptions when data changes frequently.
- `instrumentation.ts` is the official integration point for observability tooling.

## What the Current Docs Get Right

### 1. App Router as the base model

The architecture and plan documents consistently assume App Router. This matches current `Next.js` guidance and is the right basis for a platform UI with mixed server rendering, streaming, and controlled mutations.

### 2. Server Components by default

The docs instruct that story, run, review, and traceability pages should use Server Components by default. This is aligned with `Next.js`, where pages and layouts in the `app` directory are Server Components by default.

### 3. Client Components only for interactivity

The docs reserve Client Components for approval controls, live logs, uploads, and diff interactions. That is the correct pattern. Official docs confirm Client Components should be used when interactivity or browser APIs are required.

### 4. Server Actions for same-origin mutations

The docs recommend Server Actions for same-origin form mutations initiated by `platform-web`. This is valid and aligned with current `Next.js` usage.

### 5. Route Handlers for explicit HTTP boundaries

The docs say Route Handlers should be used for explicit HTTP needs such as streaming, artifact transfer, and callbacks. This is consistent with the official file conventions for Route Handlers.

### 6. Dynamic treatment of operational pages

The docs describe run status, logs, approvals, and traceability as dynamic operational views rather than static content. This is correct. `Next.js` documentation is explicit that static generation is a poor fit for pages whose content changes frequently or per request.

### 7. Framework-level telemetry hook

The docs reference `instrumentation.ts` for framework-level telemetry. This is directly supported by the official file convention.

## Corrections and Tightening Needed

### 1. Do not overstate Server Actions as a replacement for the platform API boundary

The architecture docs are mostly careful, but this line needs to stay very clear:

- Server Actions are a UI-side mutation mechanism
- they are not a reason to bypass the platform’s validation, authorization, audit, and idempotency rules

This matters because the platform design also requires a stable command/query/streaming/artifact boundary.

Recommendation:

- document that Server Actions must call shared server-side application services that preserve the same invariants as the public platform API

### 2. Route Handlers should remain HTTP-specific, not the default data layer

The current docs already lean this way, which is good.

Recommendation:

- keep Route Handlers limited to actual HTTP surfaces such as:
  - streaming endpoints
  - uploads and downloads
  - artifact access
  - external callbacks
- avoid using Route Handlers as a generic internal data-fetch layer for server-rendered pages

### 3. Be more explicit about dynamic rendering choices

The docs correctly say operational views are dynamic, but implementation needs a concrete rule.

In App Router, this can be expressed through:

- `fetch(..., { cache: 'no-store' })` for truly live server-fetched data
- route segment config such as `dynamic`
- selective revalidation where some views can tolerate bounded staleness

Recommendation:

- document per-screen freshness expectations instead of relying on a blanket statement about “dynamic operational views”

### 4. SSE is valid, but implementation should be validated at the Route Handler level

The docs prefer `Server-Sent Events (SSE)` for MVP streaming. This is reasonable for a local-first operational UI.

Recommendation:

- keep SSE under Route Handlers or another explicit HTTP boundary
- define reconnect and sequencing behavior at the platform contract level, not only at the UI level

## Implementation Guidance

Based on the validation, `Next.js` is a strong fit if implementation follows these rules:

1. Use App Router throughout the platform UI.
2. Keep pages and layouts server-first unless the browser truly needs control.
3. Use Client Components only for interactions, browser APIs, or live UI state.
4. Use Server Actions for same-origin commands initiated by forms and controls.
5. Use Route Handlers only where an HTTP surface is actually required.
6. Treat logs, run state, approvals, and traceability as dynamic data with explicit caching rules.
7. Register telemetry and request correlation through `instrumentation.ts`.

## Result

Validation result: `pass with refinements`

`Next.js` is a valid match for the platform UI architecture described in `/docs`. The current design is largely correct, with the main refinement being to keep Server Actions, Route Handlers, and the platform API boundary clearly separated in implementation guidance.
