# Skill 014: Diagnose Test Failures Using Execution Traffic

## 1) Skill Name
Diagnose SOAtest execution failures using runtime traffic + XML results.

## 2) Objective
Run a targeted server-side test, correlate XML violations with traffic request/response evidence, status-code expectation semantics, service-definition context, and chained-tool behavior, then produce a first-pass root-cause diagnosis.

This is a **general post-execution diagnostics skill** and should be applied whenever a run needs deeper failure analysis.

## 3) Scope
- In scope:
  - targeted execution using `soatestOptions.testNames`
  - collect runtime artifacts (`status`, `results`, decoded XML, traffic)
  - map failure signals to request/response evidence
  - inspect REST Client configuration relevant to pass/fail behavior
  - inspect chained validation tools for downstream failure contributors
  - retrieve service definition context by default when configured (for example OpenAPI/Swagger)
  - produce diagnosis starter notes (what failed, where, likely cause)
- Out of scope:
  - editing `.tst` assets
  - auto-remediation of test logic
  - environment/data fixes beyond diagnosis

## 4) Inputs
- Required:
  - execution config key (`general.config`)
  - target `.tst` resource path
  - target test name for focused execution
  - test suite `entityId` for traffic discovery
- Optional:
  - strict timeout / polling interval
  - additional evidence extraction from decoded XML
  - target REST Client id (for direct config inspection)
  - expected service-definition source URL (if known)

## 5) Preconditions
- API reachable and authenticated.
- Execution config exists in workspace.
- Target `.tst` and target test name exist.
- Naming policy enforced enough to avoid `testNames` ambiguity.
- If spec-aware diagnosis is needed, the service definition URL is reachable from the runtime environment.
- If service-definition URLs are parameterized, environment-variable resolution data is available.

## 6) Procedure
1. Queue focused run:
   - `POST /v6/testExecutions?now=true`
   - include:
     - `general.showdetails=true`
     - `scopeOptions.workspace.resources=["<tst-path>"]`
     - `soatestOptions.testNames=[{"value":"<exact-test-name>","match":true}]`
2. Poll completion:
   - `GET /v6/testExecutions/{id}/status` until `isRunning=false`.
3. Fetch results + decode XML:
   - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
   - decode `xmlReport` base64 to `.xml`.
4. Fetch traffic viewers from suite context:
   - `GET /v6/testExecutions/{id}/traffic?entityId=<test-suite-id>`
5. Fetch target traffic viewer payload:
   - pick relevant `trafficViewers[].id`
   - `GET /v6/testExecutions/{id}/traffic?entityId=<traffic-viewer-id>`
  - use the specific tool's traffic viewer id for precise diagnostics (for example DB Tool viewer vs REST Client viewer)
  - if selected viewer has empty `toolSettings.testRuns`, retry with other suite-returned viewer ids before escalating.
  - if all viewer payloads are empty after a detail-enabled run, extract request/response evidence from decoded XML report (`TrafficData`) and continue diagnosis with explicit evidence-source labeling.
  - do not add an automatic rerun step solely because viewer payloads remain sparse once the focused run already included `general.showdetails=true`.
6. Inspect REST Client configuration context:
  - `GET /v6/tools/restClients?id=<rest-client-id>`
  - capture at minimum:
    - request target URL/method
    - service-definition linkage (if configured)
    - `misc.validHttpResponseCodes`
  - interpret `misc.validHttpResponseCodes` as follows:
    - empty value (`""`) => default expected success code `200`
    - single value (for example `404`) => only that code is expected
    - range (for example `500-599`) => any code in range is expected
    - comma-separated set/ranges (for example `302, 500-599`) => union of all listed expectations
7. Inventory chained tools off REST Client output:
  - use `restClientsResponse.relationships.childrenRel` as primary chain inventory (for example JSON Assertor, JSON Validator, Diff Tool)
  - if needed, follow each child `url` to inspect exact tool configuration
  - inspect tool configs where available:
    - `GET /v6/tools/jsonAssertors?id=...`
    - `GET /v6/tools/jsonValidators?id=...`
    - `GET /v6/tools/diffTools?id=...`
8. Spec-aware comparison (default when configured):
  - if REST Client references a service definition URL, fetch that document.
  - if REST Client resource linkage is indirect, inspect decoded XML `APICoverage/service[@definition-uri]` as fallback source for spec URL.
  - if URL uses environment variables, resolve variables from active environment context before fetch.
  - compare request path/method and expected status/response schema with observed traffic.
9. Correlate and diagnose:
  - from XML: failing test names, violation messages, categories
  - from traffic: request path/headers + response code/body
  - when a detail-enabled traffic read is still empty, use XML-report traffic fallback evidence and mark confidence accordingly
  - from REST Client config: whether observed status code is considered valid by `validHttpResponseCodes` (empty/default means expected `200`)
  - from spec (when configured): whether observed status and payload type match API contract
  - from chained tools: whether failure is transport/status, payload shape/content, or chained assertion/validation mismatch
  - produce concise diagnosis starter with probable cause and next check.

## 6.1) Execution Analysis Triad (Reusable Default)
Treat the following as a default run-analysis sequence:
1. Execute (`POST /v6/testExecutions`, poll `/status`).
2. Read results (`/results?includeXmlReport=true`) for summary + violations.
3. Read traffic (`/traffic?entityId=...`) for concrete request/response proof.

Do not finalize root-cause conclusions until all three are collected, unless one step is unavailable.

## 7) Validation
- Expected HTTP status codes (API transport only):
  - queue/status/results/traffic/config reads typically return `200` on successful API calls
  - non-`200` from tool-under-test is diagnostic input, not automatically test failure by itself
- Expected response shape:
  - results include `summary.failureCount` and `xmlReport`
  - traffic includes `trafficViewers[]` and `toolSettings.testRuns[]`
- Post-condition checks:
  - decoded XML contains functional violations for failed tests
  - focused diagnostics execution includes `general.showdetails=true`
  - traffic response captures target request/response pair
  - REST Client config is captured (including `misc.validHttpResponseCodes`)
  - status-code expectation set is explicitly computed from `validHttpResponseCodes`
  - service-definition comparison is included when configured
  - chained tools (if present) are inventoried and included in diagnosis notes

## 8) Failure Modes
- `400` queue payload errors (missing scope/config).
- `404` unknown config/job id.
- `400` traffic `entityId` wrong type (`.tst` id or REST Client test id).
- `400` traffic query uses wrong key; parameter must be named `entityId`.
- Empty `testRuns` in non-executed viewers (not necessarily an error).
- Focused diagnostics run launched without `general.showdetails=true` can make viewer payloads look misleadingly empty even when execution traffic exists.
- Empty `testRuns` across all viewers after a detail-enabled run can hide primary evidence if XML-report fallback is skipped.
- Misclassification risk: treating HTTP status alone as root cause without checking `validHttpResponseCodes` and chained-tool expectations.
- Misclassification risk: treating non-200 as failure when configuration allows that code/range (for example `302, 500-599`).
- Spec drift risk: service definition expects one status/schema, runtime service returns another.
- Cascade risk: upstream non-JSON/plain-text response triggers downstream JSON Assertor/Validator failures that are secondary, not primary.

## 9) Safety / Rollback
- Read-only on test assets.
- No workspace mutation required.
- Optional cleanup: remove run artifacts folder if needed.

## 10) Reuse Notes
- Primary target: SOAtest.
- Not applicable in Virtualize.
- Depends on:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
- Related tool APIs used for holistic diagnosis:
  - `/v6/tools/restClients`
  - `/v6/tools/jsonAssertors`
  - `/v6/tools/jsonValidators`
  - `/v6/tools/diffTools`

## 11) Validation Evidence Pattern (General)
- This skill is general-purpose. Validation snapshots are examples, not requirements.
- For each run, capture:
  - execution metadata (`job id`, summary counts)
  - XML violations (primary/secondary failure signals)
  - target traffic request/response payloads
  - REST Client config (`validHttpResponseCodes`, endpoint/method, service-definition linkage)
  - computed effective status expectations (after parsing `validHttpResponseCodes`)
  - spec URL and comparison notes when configured
  - chained tool inventory and salient assertions/validators
  - diagnosis note with confidence and next check

## 12) Example Snapshot (Non-Normative)
- Focused run example only:
  - test name: `Simple REST Client - Test In Root Test Suite`
  - job id: `1795602334`
  - summary: `testRunCount=3`, `failureCount=4`
- Example evidence:
  - request: `GET /parabank/services/bank/customers/12213/accounts`
  - response: `HTTP/1.1 400`, payload `Could not find customer #12213`
  - XML violation includes `Received HTTP Response Code 400` and `Invalid JSON...`
- Example diagnosis starter:
  - probable setup/data mismatch for customer `12213`, with downstream JSON parsing/assertion fallout.
- Example artifact folder:
  - `work/runs/2026-03-03/tst-current/test-failure-diagnostics-root-rest/`
