# AI1 Platform Agent Contracts

## Purpose

This document defines strict input and output contracts for each specialist agent in the AI1 Platform, following [plan.md](c:/Code/AI1-Platform/docs/plan.md).

These contracts are intended to be enforced by:

- `platform-api`
- `workflow-worker`
- agent workers
- validation and review gates

They are aligned with:

- [schemas.md](c:/Code/AI1-Platform/docs/schemas.md)
- [temporal-workflows.md](c:/Code/AI1-Platform/docs/temporal-workflows.md)
- [architecture.md](c:/Code/AI1-Platform/docs/architecture.md)

## Shared Rules

All agents must be:

- narrow in responsibility
- deterministic where possible
- tool-driven
- constrained by schemas and contracts

All agents must work through platform-managed artifacts and workflow state. Agents do not talk directly to each other.

## Shared Agent Run Contract

Every agent run must receive:

- `runId`
- `storyId`
- `agentType`
- `taskIds`
- `allowedTools`
- `inputArtifacts`
- `contextScope`
- `outputSchemaVersion`
- `policyConstraints`

Every agent run must return:

- `status`
- `summary`
- `resultPayload`
- `artifactReferences`
- `changedFiles`
- `assumptions`
- `blockers`
- `confidence`
- `reviewNotes`

## Shared Input Envelope

Canonical shape:

```json
{
  "runId": "RUN-123",
  "storyId": "ST-1024",
  "agentType": "frontend",
  "taskIds": ["FE-1"],
  "allowedTools": ["read-files", "search-code", "write-files", "run-tests"],
  "inputArtifacts": [
    {
      "artifactId": "ART-1",
      "artifactType": "architecture-doc",
      "path": "artifacts/stories/ST-1024/architecture.md"
    }
  ],
  "contextScope": {
    "repo": "platform-web",
    "modules": ["auth", "ui/login"],
    "storyScopedOnly": true
  },
  "outputSchemaVersion": "1.1.0",
  "policyConstraints": {
    "approvalRequiredForSchemaChange": true,
    "allowedPaths": ["apps/platform-web/**"],
    "forbiddenActions": ["invent-unapproved-api"]
  }
}
```

## Shared Output Envelope

Canonical shape:

```json
{
  "status": "completed",
  "summary": "Implemented forgot-password UI flow.",
  "resultPayload": {},
  "artifactReferences": [
    {
      "artifactId": "ART-22",
      "artifactType": "patch",
      "path": "artifacts/runs/RUN-123/frontend.patch"
    }
  ],
  "changedFiles": ["apps/platform-web/app/login/page.tsx"],
  "assumptions": [],
  "blockers": [],
  "confidence": 0.87,
  "reviewNotes": []
}
```

## Status Contract

Allowed output statuses for all agents:

- `queued`
- `running`
- `blocked`
- `failed`
- `completed`
- `needs-review`

Status semantics:

- `blocked`: progress cannot continue without clarification, approval, or external change
- `failed`: execution failed unexpectedly or violated runtime assumptions
- `needs-review`: work completed but requires policy or human validation before downstream use

## Tasks Agent

### Purpose

Convert story inputs into executable work packages without inventing detailed solution design.

### Allowed Inputs

- `Story`
- `AcceptanceCriterion[]`
- design notes, mockups, and references
- repo standards and repo structure context
- prior blueprint patterns only when explicitly supplied as context

### Required Input Artifacts

- story record artifact or payload
- acceptance criteria payload
- coding standards or contribution guide references where available

### Forbidden Inputs

- unapproved architecture decisions as source of truth
- speculative APIs not present in current contracts

### Required Output Payload

The tasks agent must return a payload shaped like:

```json
{
  "storyId": "ST-1024",
  "normalizedStory": {
    "storyId": "ST-1024",
    "title": "User can reset password"
  },
  "tasks": [
    {
      "taskId": "FE-1",
      "storyId": "ST-1024",
      "agentType": "frontend",
      "title": "Add forgot password form",
      "description": "Add login-adjacent password reset request UI.",
      "status": "proposed",
      "definitionOfDone": ["form renders", "submission handled"],
      "dependsOn": []
    }
  ],
  "dependencies": [
    {
      "taskId": "TA-1",
      "dependsOnTaskId": "FE-1",
      "dependencyType": "hard"
    }
  ],
  "definitionOfDone": ["all acceptance criteria mapped to tasks"],
  "predictedImpactedFiles": ["apps/platform-web/app/login/page.tsx"],
  "riskFlags": [],
  "ambiguityFlags": []
}
```

### Required Output Artifacts

- task graph artifact
- dependency graph artifact
- ambiguity and risk report
- handoff packet for architect and delivery agents

### Required Guarantees

- every acceptance criterion is mapped to at least one task
- tasks are non-overlapping where possible
- dependencies are explicit
- unknowns are surfaced, not hidden

### Forbidden Behavior

- inventing architecture
- inventing schema changes
- inventing backend APIs
- marking ambiguity as resolved without evidence

## Architect Agent

### Purpose

Produce the technical blueprint and the machine-readable contracts that govern implementation.

### Allowed Inputs

- `Story`
- `AcceptanceCriterion[]`
- `Task[]`
- existing repo architecture
- current APIs
- current data model definitions
- repo context from `context-service`

### Required Input Artifacts

- task decomposition artifact
- story and acceptance criteria artifacts
- relevant repo context references

### Forbidden Inputs

- implementation patches as primary design source
- unverifiable assumptions presented as approved fact

### Required Output Payload

The architect agent must return a payload shaped like:

```json
{
  "storyId": "ST-1024",
  "blueprint": {
    "blueprintId": "BP-1024",
    "storyId": "ST-1024",
    "sourceRunId": "RUN-ARCH-1",
    "version": 1,
    "status": "draft",
    "summary": "Password reset uses existing auth service and new reset request endpoint.",
    "componentBoundaries": [],
    "dataFlows": [],
    "implementationSequence": []
  },
  "apiContracts": [],
  "dataModelChanges": [],
  "impactAnalysis": {
    "services": ["platform-api"],
    "modules": ["auth"],
    "risks": []
  },
  "approvalFlags": {
    "requiresArchitectureApproval": true,
    "reasons": ["new-api"]
  }
}
```

### Required Output Artifacts

- `architecture.md`
- `api-contract.json`
- `data-model.json`
- `sequence-flow.md` or equivalent structured flow artifact
- `impact-analysis.json`

### Required Guarantees

- contracts are machine-readable
- architecture decisions are explicit
- schema and API impacts are explicit
- implementation sequencing is actionable
- approval-required changes are flagged

### Forbidden Behavior

- writing production code as the main output
- skipping machine-readable contracts
- approving its own high-risk design changes

## Front-End Agent

### Purpose

Implement UI and client-side behavior against approved architecture and contracts.

### Allowed Inputs

- front-end `Task[]`
- approved `ArchitectureBlueprint`
- approved `ApiContract`
- design-system guidance
- existing front-end code context

### Required Input Artifacts

- approved blueprint artifact
- approved API contract artifact
- front-end task artifact
- file ownership or allowed path policy

### Forbidden Inputs

- draft or unapproved backend contracts when approval is required
- paths outside assigned ownership scope

### Required Output Payload

The front-end agent must return a payload shaped like:

```json
{
  "storyId": "ST-1024",
  "runId": "RUN-FE-1",
  "changeSet": {
    "changeSetId": "CS-FE-1",
    "agentType": "frontend",
    "summary": "Added forgot password UI and request submission flow.",
    "changedFiles": [
      {
        "path": "apps/platform-web/app/login/page.tsx",
        "changeType": "modified"
      }
    ]
  },
  "tests": ["apps/platform-web/tests/forgot-password.spec.ts"],
  "contractMismatches": [],
  "assumptions": [],
  "blockers": []
}
```

### Required Output Artifacts

- patch artifact
- changed file manifest
- component or integration test artifacts where relevant
- change summary artifact

### Required Guarantees

- only approved contracts are implemented
- changed files stay within allowed ownership boundaries
- contract mismatches are emitted immediately
- generated code remains traceable to story and task IDs

### Forbidden Behavior

- inventing backend APIs
- silently deviating from the architect contract
- editing files outside approved scope

## Back-End Agent

### Purpose

Implement service, API, and data-layer changes against approved architecture.

### Allowed Inputs

- back-end `Task[]`
- approved `ArchitectureBlueprint`
- approved `ApiContract`
- approved `DataModelChange[]`
- existing backend code and schema definitions

### Required Input Artifacts

- approved blueprint artifact
- approved API contract artifact
- approved data model change artifact where relevant
- backend task artifact

### Forbidden Inputs

- unapproved schema changes
- UI-layer requirements that contradict approved architecture

### Required Output Payload

The back-end agent must return a payload shaped like:

```json
{
  "storyId": "ST-1024",
  "runId": "RUN-BE-1",
  "changeSet": {
    "changeSetId": "CS-BE-1",
    "agentType": "backend",
    "summary": "Added password reset request endpoint and service wiring.",
    "changedFiles": [
      {
        "path": "apps/platform-api/src/modules/auth/routes.ts",
        "changeType": "modified"
      }
    ]
  },
  "migrations": [],
  "tests": ["apps/platform-api/src/modules/auth/routes.test.ts"],
  "updatedApiContractArtifactId": "ART-API-2",
  "stubs": [],
  "assumptions": [],
  "blockers": []
}
```

### Required Output Artifacts

- patch artifact
- changed file manifest
- migration artifact when required
- unit or contract test artifacts
- updated API contract or stubs where relevant

### Required Guarantees

- contract validation is preserved
- schema changes are only implemented when approval gates are satisfied
- front-end and test consumers receive contract-stable outputs

### Forbidden Behavior

- bypassing contract validation
- introducing unapproved schema changes
- silently changing API behavior without updating contract artifacts

## Test-Automation Agent

### Purpose

Map acceptance criteria to executable `Playwright` coverage and supporting test artifacts.

### Allowed Inputs

- `Story`
- `AcceptanceCriterion[]`
- approved `ArchitectureBlueprint`
- front-end and back-end outputs
- seeded test data strategy

### Required Input Artifacts

- acceptance criteria artifact
- approved blueprint artifact
- delivery run outputs or approved interface artifacts
- environment assumptions where known

### Forbidden Inputs

- incomplete acceptance criteria
- undocumented environment dependencies treated as fixed

### Required Output Payload

The test-automation agent must return a payload shaped like:

```json
{
  "storyId": "ST-1024",
  "runId": "RUN-TA-1",
  "testScenarios": [
    {
      "testScenarioId": "TS-1",
      "storyId": "ST-1024",
      "acceptanceCriterionId": "AC-1",
      "automationStatus": "generated",
      "framework": "playwright",
      "specPath": "agents/test-agent-worker/generated/reset-password.spec.ts",
      "environmentAssumptions": []
    }
  ],
  "coverageMap": [
    {
      "acceptanceCriterionId": "AC-1",
      "testScenarioId": "TS-1"
    }
  ],
  "evidencePaths": [],
  "blockers": []
}
```

### Required Output Artifacts

- `Playwright` spec artifacts
- fixtures and helper artifacts
- acceptance-criteria coverage mapping artifact
- blocker report

### Required Guarantees

- each acceptance criterion maps to one or more test scenarios
- environment assumptions are explicit
- blockers are recorded instead of hidden
- evidence paths are captured when execution occurs

### Forbidden Behavior

- declaring coverage complete without criterion mapping
- generating tests with undocumented environment assumptions
- marking blocked scenarios as passing

## Tool Contracts

Allowed tool categories by agent:

- tasks agent:
  - read files
  - search code
  - inspect repo structure
  - read schemas and contracts

- architect agent:
  - read files
  - search code
  - inspect APIs and schemas
  - write architecture artifacts

- front-end agent:
  - read files
  - search code
  - write files inside approved scope
  - run targeted tests
  - inspect diffs

- back-end agent:
  - read files
  - search code
  - write files inside approved scope
  - run targeted tests
  - inspect contracts and schemas

- test-automation agent:
  - read files
  - inspect acceptance criteria and outputs
  - write test files
  - run test generation and targeted verification

## Policy Constraints

Every agent run should carry explicit policy constraints:

- `allowedPaths`
- `forbiddenActions`
- `requiresApprovedBlueprint`
- `requiresApprovedApiContract`
- `requiresApprovedDataModelChange`
- `mustEmitContractMismatch`
- `mustRecordTraceability`

Example:

```json
{
  "allowedPaths": ["apps/platform-web/**"],
  "forbiddenActions": ["invent-unapproved-api", "edit-backend-files"],
  "requiresApprovedBlueprint": true,
  "requiresApprovedApiContract": true,
  "requiresApprovedDataModelChange": false,
  "mustEmitContractMismatch": true,
  "mustRecordTraceability": true
}
```

## Failure Contract

If an agent cannot complete work, it must return structured failure information instead of vague prose.

Required failure fields:

- `status`
- `summary`
- `blockers`
- `assumptions`
- `reviewNotes`

Blocked example:

```json
{
  "status": "blocked",
  "summary": "Front-end task cannot proceed because the approved API contract does not include the required endpoint.",
  "blockers": [
    "Missing approved POST /auth/password-reset endpoint in api-contract.json"
  ],
  "assumptions": [],
  "reviewNotes": [
    "Route to architecture or backend contract update before retrying frontend implementation."
  ]
}
```

## Enforcement Expectations

The orchestrator should reject or block runs when:

- required input artifacts are missing
- the wrong approval state is supplied
- output payloads fail schema validation
- changed files fall outside `allowedPaths`
- required traceability fields are missing
- contract mismatch is detected but not reported

## Why These Contracts Fit The Plan

- They preserve narrow agent responsibilities.
- They enforce contracts over prose.
- They align agent outputs with the workflow stages in the orchestrator.
- They make human approval and deterministic validation enforceable.
- They provide enough structure for `platform-api`, `workflow-worker`, and reviewers to treat agent output as machine-governed artifacts instead of informal text.
