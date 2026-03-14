# Skill 047: Generated Service-Definition Subset Prune

## 1) Skill Name
Prune generated suites/tests to a user-selected operation subset.

## 2) Objective
After creating a `.tst` from a full service definition, remove non-selected operation suites so final scope matches confirmed user intent.

## 3) Scope
- In scope:
  - identify generated operation suites under service-definition root suite
  - delete non-selected operation suites via suite delete endpoint
  - verify remaining scope exactly matches selected operations
- Out of scope:
  - generating tests from service definition
  - enriching kept tests with negative variants/tooling
  - semantic selection design (owned by orchestration cards)

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`

## 4) Inputs
- Required:
  - target `.tst` id
  - service-definition root suite id (for example `/.../openapi.yaml`)
  - selected operation suite names (exact match list)
- Optional:
  - dry-run mode (report only, no delete)

## 5) Preconditions
- API reachable and authenticated.
- Target `.tst` exists and has generated operation suites.
- Selected operation list has been user-confirmed.
- Child type metadata is available from `GET /v6/children` (for example `type`).

## 6) Procedure
1. Enumerate direct children under service-definition root suite:
   - `GET /v6/children?id=<service-definition-root-suite-id>`
2. Partition child suites:
  - candidate operation suites: children where `type=testSuite`
  - keep list: candidate suite names in selected operation list
  - prune list: candidate suite names not selected
  - do not include non-`testSuite` children (for example `environment`, `testCase`, `outputTool`) in prune candidates.
3. Delete each prune suite:
   - `DELETE /v6/suites?id=<suite-id>`
4. Re-read children under same root suite.
5. Verify post-condition:
   - all kept suite names remain
   - no pruned suite names remain
   - no extra operation suites remain.

## 7) Validation
- Expected HTTP status codes:
  - `200` on successful suite delete
  - `404` invalid suite id
  - `405` wrong endpoint for delete (indicates endpoint mismatch)
- Post-condition checks:
  - remaining operation suite set equals selected scope set.

## 8) Failure Modes
- Using incorrect delete endpoint (`/v6/suites/testSuites`) yields `405`.
- Pruning parent service-definition suite by mistake instead of child operation suites.
- Name-matching mismatch when selected operation strings are not exact.
- Attempting to delete non-suite nodes (for example `Environment`) via suite delete endpoint.

## 8.1 Type Safety Guard (Required)
- Before delete, assert each prune candidate satisfies all of:
  - `type == testSuite`
  - id is a direct child returned for the targeted service-definition root suite in the current read.
- If any candidate is non-`testSuite`, skip delete for that node and log a warning; continue with valid suite candidates only.
- If zero valid `testSuite` candidates remain after filtering, perform no deletes and return a no-op prune outcome.

## 9) Safety / Rollback
- Write operation (deletes suites/tests under target `.tst`).
- Rollback options:
  - recreate `.tst` from service definition and reapply prune correctly,
  - or restore from pre-prune copied backup `.tst`.

## 10) Reuse Notes
- Primary target: SOAtest.
- Not applicable in Virtualize.
- Typical dependency chain:
  - generation skill (022/023/024/025) -> this prune skill -> variant/tooling skills.
