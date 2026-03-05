# Skill 009: Test Suite Creation & Configuration

## 1) Skill Name
Test suite creation and configuration (staged validation)

## 2) Objective
Create a SOAtest test suite and configure its key options in controlled stages with explicit verification per option group.

## 3) Scope
- In scope:
  - create suite using `POST /v6/suites/testSuites`
  - update suite fields using `PUT /v6/suites/testSuites?id=...`
  - verify configuration through `GET /v6/suites/testSuites?id=...`, `GET /v6/children`, and optional YAML download
  - staged rollout for large option surface
- Out of scope (for initial pass):
  - wsdl-generated suite creation path (`/v6/suites/testSuites/wsdl`)
  - end-to-end execution behavior validation of all options
  - policy/semantic conventions (covered in higher layers)

## 4) Inputs
- Required:
  - file/suite parent id (top-level suite parent)
  - target suite name
- Optional:
  - initial `disabled` state
  - requirements/notes
  - execution options
  - suite variables
  - reference settings

## 5) Preconditions
- API base URL reachable and authenticated.
- Target parent id exists and is writable.
- Rollback snapshot plan is available for write tests.

## 6) Procedure
1. Preflight parent and naming check via `GET /v6/children?id=<parent-id>`.
2. Create suite with minimal payload using `POST /v6/suites/testSuites` (`parent`, `name`).
3. Verify create response (`id`, `name`, `url`) and presence under parent children.
4. Apply option updates in small batches via `PUT /v6/suites/testSuites?id=<suite-id>`.
5. After each batch, read back suite config and confirm only intended fields changed.
6. Capture before/after artifacts and update run report.

## 7) Validation
- Expected HTTP status codes:
  - `200` success for create/update/read
  - `400` invalid payload/fields
  - `403/401` auth/license constraints
- Expected response shape:
  - suite responses include `id`, `url`, and configuration fields under suite payload
- Post-condition checks:
  - created suite exists at expected parent
  - configured fields persist across re-read
  - unrelated fields remain stable unless intentionally set

## 8) Failure Modes
- Parent id is not a valid top-level suite parent.
- Duplicate name collisions in same parent context.
- PUT payload omits fields and defaults override unintentionally.
- API response accepts payload but UI/state lags until refresh.

## 9) Safety / Rollback
- Read-only by default? No (write skill).
- Rollback plan:
  - for isolated tests, create a temporary suite and delete it after validation
  - for existing suite edits, restore from pre-change YAML snapshot if needed

## 10) Reuse Notes
- Applies to SOAtest: Yes.
- Applies to Virtualize: Not validated.
- Shared components involved:
  - `POST /v6/suites/testSuites`
  - `PUT /v6/suites/testSuites?id=...`
  - `GET /v6/suites/testSuites?id=...`
  - `GET /v6/children`
  - optional: `GET /v6/files/download`, `POST /v6/files/upload`
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - do not keep default generated names; enforce unique deterministic names when creating new tests/tools.

## 11) Deferred Option Matrix (to validate later)
- Core identity:
  - `name`, `disabled`
- Notes/requirements:
  - `requirementsAndNotes`
- Variables:
  - suite `variables` add/update/remove
- Execution:
  - `executionOptions`
- Reference mode:
  - `referenced`, `referenceLocation`

## 12) Next Validation Pass Plan
1. Minimal create/delete smoke test.
2. Name + disabled update test.
3. Variables add/update/remove test.
4. Execution options update test.
5. Reference mode toggle test.
6. Consolidated regression readback + YAML confirmation.
