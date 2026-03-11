# ChromaDB Validation

## Scope

This document validates the AI1 Platform design assumptions about `ChromaDB` against the best available Chroma documentation surfaced through Context7.

Context7 match:

- Library ID: `/websites/cookbook_chromadb_dev`

Validated on:

- March 11, 2026

Source basis:

- Chroma cookbook documentation retrieved through Context7 for persistent local clients, collections, metadata, and filtered retrieval

## Validation Confidence

This validation has lower source confidence than the `Temporal`, `Next.js`, `React`, `TypeScript`, `Node.js`, and `SQLite` validations.

Reason:

- Context7 surfaced the Chroma cookbook as the best available match rather than a clearly canonical core product documentation source

The conclusions below are still useful, but they should be treated as `provisionally validated` until cross-checked against the latest primary Chroma product documentation during implementation.

## Validation Summary

The current documentation in `/docs` is directionally aligned with `ChromaDB` as a local vector retrieval layer for contextual artifacts.

The following assumptions are supported by the available docs:

- `ChromaDB` supports persistent local storage.
- It organizes data through collections.
- It supports metadata attached to stored documents/embeddings.
- It supports filtered retrieval using metadata filters.
- It is appropriate for scoped retrieval scenarios over indexed document sets.

## What the Current Docs Get Right

### 1. ChromaDB as a retrieval layer, not the transactional source of truth

The platform docs position `ChromaDB` as a vector/context store while keeping authority in:

- `SQLite` for structured state
- filesystem or artifact storage for approved files and large outputs

That is the right architectural separation.

### 2. Local-first storage model

The docs use `ChromaDB` in a local-first way. The available documentation confirms support for a persistent local client that stores data on disk.

### 3. Scope-first retrieval

The platform docs emphasize scoped retrieval rather than whole-repo dumping. The Chroma documentation supports metadata-based filtering, which fits this design well.

### 4. Indexing contextual artifacts

The docs propose indexing:

- repository docs
- architecture artifacts
- coding standards
- prior runs
- API contracts
- design system guidance

This is a sensible use case for a vector retrieval store with metadata filters and collection organization.

## Corrections and Tightening Needed

### 1. Be explicit that ChromaDB is retrieval infrastructure, not canonical data storage

The docs already lean this way, which is correct.

Recommendation:

- state explicitly that Chroma results are retrieval candidates only
- keep business authority in approved artifacts and relational state

### 2. Define the retrieval scoping model concretely

The docs talk about scope-first retrieval, but implementation should make the scoping fields explicit.

Recommendation:

- define metadata dimensions such as:
  - `storyId`
  - `runId`
  - `agentType`
  - `artifactType`
  - `repo`
  - `module`
  - `documentKind`
  - `schemaVersion`

This makes filtered retrieval operational rather than aspirational.

### 3. Define collection strategy early

The available documentation confirms collections are a core organizational unit, but the platform docs do not yet define how they should map to platform concerns.

Recommendation:

- choose a clear strategy such as:
  - one collection per repository
  - one collection per document class
  - one collection per environment with metadata filters for finer scoping

Do not let collection design emerge accidentally.

### 4. Embedding and re-indexing strategy is still underspecified

The current platform docs mention indexing, but not enough implementation detail exists yet around:

- chunking rules
- embedding model choice
- re-index triggers
- stale embedding cleanup
- artifact version invalidation

Recommendation:

- add an explicit indexing lifecycle before implementation starts

### 5. Retrieval quality must remain subordinate to hard constraints

Vector similarity is useful but inherently probabilistic.

Recommendation:

- never let Chroma retrieval override:
  - approved architecture contracts
  - current source code
  - typed API contracts
  - validated workflow state

## Implementation Guidance

Based on the validation, `ChromaDB` is a plausible fit if implementation follows these rules:

1. Use `ChromaDB` only for retrieval and context discovery.
2. Keep `SQLite` and approved artifacts as the authoritative system of record.
3. Use persistent local storage for local-first development and execution.
4. Design retrieval around metadata filters and explicit scope boundaries.
5. Define collection strategy, chunking, embedding model choice, and re-index behavior before large-scale ingestion.
6. Record retrieval provenance so runs can explain which context documents were used.

## Result

Validation result: `pass with caution`

`ChromaDB` appears to be a reasonable fit for the AI1 Platform retrieval layer described in `/docs`, and the current design is directionally sound. The main caution is that this validation is based on the Chroma cookbook surfaced by Context7 rather than a stronger canonical documentation source, so implementation should verify operational details against current primary Chroma docs before locking the design.
