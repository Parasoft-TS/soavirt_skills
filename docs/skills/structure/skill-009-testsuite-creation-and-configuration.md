# Skill 009: Test Suite Lifecycle (CRUD + Configuration Hardening)

## 1) Skill Name
Test suite create/read/update/delete/copy/move/reorder with option-safe configuration.

## 2) Objective
Manage SOAtest test suites as first-class lifecycle entities (not create-only), while safely configuring common and less-common suite options with explicit readback verification.

## 3) Scope
- In scope:
  - create suite via `POST /v6/suites/testSuites`
  - read suite via `GET /v6/suites/testSuites?id=...`
  - update suite via `PUT /v6/suites/testSuites?id=...`
  - delete suite via `DELETE /v6/suites?id=...`
  - copy/move suite via `POST /v6/suites/copy`, `POST /v6/suites/move`
  - reorder immediate children via `PUT /v6/suites/children?id=...`
  - verify via `GET /v6/children` and optional YAML download
- Out of scope:
  - WSDL-generated suite creation path (`POST /v6/suites/testSuites/wsdl`)
  - execution-level semantic test design inside child tools/tests

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`
- Additive:
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`

## 4) Inputs
- Required:
  - target parent id (for create/copy/move destination)
  - suite name (for create; optional for rename update)
- Optional:
  - `disabled`
  - `requirementsAndNotes`
  - `variables[]`
  - `executionOptions`
  - `executionOptions.testExecution.executionMode` (`sequential` or `concurrent`)
  - `referenced`, `referenceLocation`
  - reorder target index/order
  - copy/move destination name override

### 4.1) Input Gate (Required)
- Never infer suite options from prior examples/runs.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context explicitly confirmed by the user.
- If required fields for requested operation are missing, solicit them in one prompt before mutation.

### 4.2) Conditional Required Matrix
- Create:
  - `parent.id`
  - `name`
- Update:
  - suite `id` plus explicit change intent (at least one field to mutate)
- Delete:
  - suite `id`
- Copy/Move:
  - `from.id`
  - `to.parent.id`
  - optional `to.name`
- Reorder:
  - parent suite `id`
  - complete child id list (all immediate children, no omissions)

### 4.3) Notes Population Rule (Create-Time, Non-Blocking)
- `requirementsAndNotes.notes` is optional for suite creation.
- Do not require a dedicated user prompt solely to collect notes.
- On create:
  - if user-provided context exists, write a concise notes summary.
  - if no explicit notes were provided but create intent/context is high-confidence and unambiguous, infer a concise notes summary.
  - if context is missing or ambiguous, leave notes blank.
- Do not auto-modify notes during unrelated update operations unless user explicitly requests notes changes.

### 4.4) Requirements Tracking Tagging Rule (Root-Suite First)
- Common use case: associating suites/tests with external ids (for example Jira story/test ids).
- Preferred scope:
  - root test suite (default),
  - sibling suite tagging only when one `.tst` intentionally contains multiple independent test areas.
- Test-level tagging is supported but non-default; use only when user explicitly requests test-level granularity.
- Supported requirement types to preserve when provided:
  - `@pr`, `@req`, `@task`, `@test`
- Recommendation: default to `@req` unless user explicitly requests another type.

### 4.5) Variable Policy (String-First + Parent Inheritance Default)
- Preferred variable type is `string`.
- Best practice for this skill: define suite variables as Strings unless user explicitly requests otherwise.
- Inheritance behavior:
  - default to parent-aware variables (`useValueFromParentSuite=true`) so child suites can inherit same-name values from parent suites.
  - local-only variable mode (`useValueFromParentSuite=false`) is non-default and should be used only when user explicitly requests non-overridable local behavior.

## 5) Preconditions
- API reachable and authenticated.
- Target ids exist and are writable.
- Pre-change snapshot is available before destructive operations (update/delete/move/reorder).

## 6) Procedure
1. Resolve operation mode: create, read, update, delete, copy, move, reorder.
2. Preflight parent/suite context:
  - `GET /v6/children?id=<parent-or-suite-id>` as needed.
3. Create flow:
  - `POST /v6/suites/testSuites` with minimal required payload.
  - optional: include `requirementsAndNotes.notes` per Section 4.3.
  - API constraint: create payload supports notes, but requirements-tracking tags should be applied after create via update flow.
4. Read flow:
  - `GET /v6/suites/testSuites?id=<suite-id>`.
5. Update flow (required safety pattern):
  - `GET /v6/suites/testSuites?id=<suite-id>`.
  - merge intended edits into current suite config.
  - `PUT /v6/suites/testSuites?id=<suite-id>` with merged payload.
  - reason: omitted PUT fields may revert to defaults; avoid sparse PUT writes.
  - when requirements tagging is requested, update `requirementsAndNotes.requirementsAndTracking[]` in this step.
6. Delete flow:
  - `DELETE /v6/suites?id=<suite-id>`.
7. Copy/move flow:
  - `POST /v6/suites/copy` or `POST /v6/suites/move` with `{ from.id, to.parent.id, to.name? }`.
8. Reorder flow:
  - build full immediate child order.
  - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id`.
9. Read back and verify:
  - confirm intended changes persisted.
  - confirm unaffected fields remained stable.
10. Optional YAML confirmation:
  - use file download/readback only when API response leaves ambiguity.

## 6.1) Configuration Tiers (Common vs Less-Common)
Common options (validate first):
- `name`
- `disabled`
- `variables[]` (add/update/remove)
- `executionOptions.testExecution.executionMode`

Less-common options (explicit intent required):
- `requirementsAndNotes`
- `executionOptions.flowLogic` (suite/test flow logic, loop/condition settings)
- `referenced` and `referenceLocation`

## 6.2) Option Mutation Rules
- For `variables[]`:
  - treat variable updates as full-array intent unless user specifies targeted merge behavior.
  - confirm each variable's `name`, type/value field, and `useValueFromParentSuite`.
  - default authoring profile:
    - `type=string`
    - set `stringValue=<value>`
    - set `useValueFromParentSuite=true` unless user explicitly requests local-only behavior.
  - if non-string variable type is requested, apply it only with explicit user intent.
- For `executionOptions`:
  - common path: set/verify `testExecution.executionMode`.
  - supported execution-mode intents:
    - `sequential` -> `TESTS_RUN_SEQUENTIALLY`
    - `concurrent` -> `TESTS_RUN_CONCURRENTLY`
  - if user intent is ambiguous between sequential/concurrent, ask a targeted clarification before update.
  - verify persisted mode via suite readback after PUT.
  - less-common path (`flowLogic`) requires explicit user confirmation before update.
- For `requirementsAndNotes`:
  - only root-level suites support requirements/tracking properties; verify suite level before applying.
  - split behavior:
    - notes may be set on create (Section 4.3),
    - requirements-tracking tags should be set/maintained via update read-merge-write.
  - preserve provided requirement type/id pairs exactly; do not auto-remap types.
- For `referenced/referenceLocation`:
  - require explicit location input; do not infer reference targets.

## 7) Validation
- Expected status codes:
  - `200` success for valid operations.
  - `400` invalid payload/shape/field combinations.
  - `404` invalid id.
  - `401/403` auth/license constraints.
  - `405` indicates wrong endpoint usage for operation class.
- Post-condition checks:
  - create/copy/move: suite present under expected parent.
  - delete: suite absent in parent descendants/children.
  - update: readback reflects intended field mutations only.
  - reorder: child sequence exactly matches submitted order.

## 8) Failure Modes
- Sparse PUT reset risk: omitted fields can be defaulted by suite PUT semantics.
- Wrong delete endpoint risk (`/v6/suites/testSuites` instead of `/v6/suites`).
- Reorder drift when payload does not include all immediate children.
- Applying root-only properties to non-root suites.
- Ambiguous variable mutation intent (replace vs targeted edit) causing unintended variable loss.
- Variable inheritance mismatch when `useValueFromParentSuite` is set opposite to requested override behavior.
- Type-policy drift when non-string variable types are introduced without explicit user intent.
- Execution-mode mismatch when user-facing intent (`sequential`/`concurrent`) is not mapped to API enum values correctly.

## 9) Safety / Rollback
- Write skill (structural + configuration mutations).
- Rollback plan:
  - preserve pre-change suite GET snapshot.
  - for temporary validation suites, delete via `DELETE /v6/suites?id=...`.
  - for edit regressions, restore full suite config via merged PUT from snapshot.
  - for move/reorder mistakes, apply inverse move/order restoration from saved state.

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Related endpoints:
  - `POST/GET/PUT /v6/suites/testSuites`
  - `DELETE /v6/suites`
  - `POST /v6/suites/copy`
  - `POST /v6/suites/move`
  - `PUT /v6/suites/children`
  - `GET /v6/children`
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - enforce deterministic non-default names for created suites.
  - `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`
  - use GET -> merge -> PUT for suite updates to avoid sparse PUT regressions.
## 11) Cross-Skill Handoff for New `.tst` Creation
- For new-file creation skills (`skill-021` through `skill-025`):
  - generation/create endpoints focus on file creation and do not provide a robust create-time requirements-tracking tag surface for root-suite tagging.
  - apply requirement tags after file creation by invoking this skill's suite update flow (`GET` -> merge -> `PUT`) on the root suite.
