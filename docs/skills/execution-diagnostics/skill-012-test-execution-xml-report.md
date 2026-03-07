# Skill 012: Server Test Execution + XML Results Report

## 1) Skill Name
Run SOAtest on server and retrieve XML report.

## 2) Objective
Execute one or more `.tst` resources on the server using `POST /v6/testExecutions`, wait for completion, and fetch runtime results including `xmlReport`.

This is a **general execution-analysis skill** to be reused whenever tests are run and runtime behavior needs investigation.

## 3) Scope
- In scope:
  - queue execution job
  - poll execution status
  - retrieve execution results with `includeXmlReport=true`
  - retrieve execution traffic viewers and request/response payloads for diagnostics
  - coarse execution narrowing using `soatestOptions.testNames`
  - persist XML report artifact
- Out of scope:
  - creating/editing test execution configurations
  - semantic analysis of report contents

## 4) Inputs
- Required:
  - execution configuration key (`general.config`)
  - target resource paths (for example `/TestAssets/Basic_Test_Construction_Learning.tst`)
- Optional:
  - `general.showdetails`
  - `general.parallel`
  - `soatestOptions.testNames` (coarse name-based execution filter)
  - result toggles: `includeHtmlReport`, `includeReportArchive`

## 5) Preconditions
- API reachable and authenticated.
- At least one valid execution configuration exists in the target server workspace.
- Target `.tst` resource path exists.

## 5.1) Canonical Payload (API-First)
Endpoint: `POST /v6/testExecutions?now=true`
```json
{
  "general": {
    "config": "soatest.user://Example Configuration"
  },
  "scopeOptions": {
    "workspace": {
      "resources": ["/TestAssets/MyTest.tst"]
    }
  },
  "soatestOptions": {
    "testNames": [
      { "value": "myTestName", "match": true }
    ]
  }
}
```
Key details:
- `scopeOptions.workspace.resources` is required (not `soatestOptions.testFile`).
- `testNames[].match` is a **boolean** (`true`/`false`), not a string like `"exact"`.
- `soatestOptions.testNames` must be an array of **objects** (`{ "value": "...", "match": true }`), not an array of plain strings.
- Append `?now=true` to the URL to start execution immediately.
- `soatestOptions.testNames` is optional; omit to run all tests in the resource.

## 6) Procedure
1. Preflight resource existence:
   - `GET /v6/descendants/files`
   - verify target `.tst` is present under `scopeOptions.workspace.resources` path (parse `children` array entries).
2. Build payload:
   - `general.config=<valid-config-key>`
  - workspace default (this repo): `soatest.user://Example Configuration`
   - `scopeOptions.workspace.resources=["<resource-path>"]`
  - optional coarse filter:
    - `soatestOptions.testNames=[{"value":"<test-name>","match":true}]`
3. Start execution:
   - `POST /v6/testExecutions?now=true`
   - capture returned job id.
   - fail-fast gate (required): if POST does not return a valid execution `id` (for example `400` payload error), stop and correct payload/config first; do not call status/results/traffic for that failed attempt.
4. Poll status:
   - `GET /v6/testExecutions/{id}/status`
   - use `isRunning` as the canonical completion signal; wait until `isRunning=false`.
   - treat `percent` as informational only (not a completion gate by itself).
5. Fetch results:
   - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
6. Optional diagnostics traffic retrieval:
  - list traffic viewers from a suite context:
    - `GET /v6/testExecutions/{id}/traffic?entityId=<test-suite-id>`
  - select a returned traffic viewer `id` and fetch targeted traffic:
    - `GET /v6/testExecutions/{id}/traffic?entityId=<traffic-viewer-id>`
  - normalize to two-step retrieval even when output-provider ids are known:
    - always discover viewer ids from suite context first,
    - then fetch focused viewer payloads by returned ids.
  - viewer-selection rule:
    - prefer the specific producer's Traffic Viewer id for the target test/tool, not just the first returned viewer.
  - fallback when traffic payload is empty:
    - if selected viewer has empty `toolSettings.testRuns`, retry with other suite-returned viewer ids for the same run,
    - if all viewer payloads remain empty, extract request/response evidence from decoded `xmlReport` traffic sections (`TrafficData`) before classifying traffic as unavailable.
  - use `toolSettings.testRuns[].trafficData.request/response` for failure analysis.
7. Persist artifacts:
   - raw POST response
   - status polling trace
   - results JSON
  - extracted `xmlReport` (base64 payload) and decoded `.xml` file.
  - optional traffic diagnostics responses.

## 6.1) Execution Analysis Triad (Reusable Default)
Use this triad as the default post-run workflow for diagnostics:
1. Run tests (`POST /v6/testExecutions`, poll `/status`).
2. Retrieve results (`/results?includeXmlReport=true`) for summary + violations.
3. Retrieve traffic (`/traffic?entityId=...`) for request/response evidence.

When in doubt, complete all three steps before drawing conclusions about failure causes.

## 7) Validation
- Expected success pattern:
  - POST returns `{ "id": "..." }`
  - final status returns `isRunning=false` and `percent=100`
  - results contain non-empty `xmlReport`
- Post-condition checks:
  - `xmlReport` length > 0
  - decoded XML starts with `<?xml` and contains `<ResultsSession ...>`
  - results summary contains `execution.testRunCount` and failure/pass counts
  - when traffic endpoint returns empty runs, fallback extraction from decoded XML report is attempted before concluding evidence is unavailable.
  - if `summary.scope.totalFilesExecuted > 0` but `summary.execution.testRunCount = 0`, treat run as no-op and investigate target test executability (for example empty REST Client URL fields, disabled tools, overly restrictive config filters).

## 8) Failure Modes
- `400 Invalid payload` when `scopeOptions.workspace.resources` missing.
- `400 Invalid payload` when `soatestOptions.testNames` is provided as a string array instead of object array (`value`/`match`).
- `404 Non-existent configuration [...]` when `general.config` key is not valid in workspace.
- `404` on status/results when job id is missing/invalid.
- `400 Invalid payload` on traffic endpoint when `entityId` type is not accepted:
  - accepted: `testSuite`, `outputTool`
  - rejected (validated): `.tst` file id (`Test (.tst) File`), REST client test id (`REST Client`).
- Empty `testRuns` from viewer traffic despite execution:
  - can occur for non-target viewers or sparse run capture,
  - requires viewer reselection and XML-report fallback before concluding missing traffic evidence.

## 8.1) Known Execution-Selection Constraints
- `soatestOptions.testNames` is **coarse name-based filtering**, not suite-path addressing.
- It does **not** map to test suite boundaries.
- Duplicate test names are all selected when names match.
- Setup tests can still execute with filtered runs when they are part of suite execution flow.
- Practical mitigation: adopt unique test names across the workspace so `testNames.value` remains deterministic.

## 8.2) Known Traffic Retrieval Constraints
- Endpoint used: `GET /v6/testExecutions/{id}/traffic?entityId=...`
- `entityId` must resolve to an object typed as `testSuite` or `outputTool`.
- Query parameter key must be `entityId` (not `id` or other aliases).
- Practical retrieval pattern:
  1. call traffic with suite id to obtain `trafficViewers[]`
  2. use each `trafficViewers[].id` (Traffic Viewer output tool id) to get tool-scoped traffic
  3. for focused diagnosis, use the **specific tool's** traffic viewer id (for example DB Tool viewer vs REST Client viewer), not a generic parent id
  4. inspect `toolSettings.testRuns` for populated runs vs empty arrays.
- Validation findings:
  - suite id `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited` returned populated `trafficViewers`.
  - `.tst` id `/TestAssets/Basic_Test_Construction_Learning.tst` returned 400 with expected types `testSuite,outputTool`.
  - REST client test id `/.../Simple REST Client - Test In Root Test Suite` returned 400 with expected types `testSuite,outputTool`.

## 9) Safety / Rollback
- Read-only on workspace assets (no `.tst` mutation required).
- Optional cleanup: `DELETE /v6/testExecutions/{id}` for queued jobs not yet started.

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Shared dependencies:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
## 11) Practical Notes
- In this environment, valid `general.config` keys were discoverable from:
  - workspace metadata file `ConfigManager.properties`
  - active value example: `soatest.user://Example Configuration`
- Default execution guidance for this workspace:
  - use `general.config=soatest.user://Example Configuration` unless user explicitly requests another known-valid config.
- The API spec does not expose a dedicated endpoint for listing execution configuration keys.