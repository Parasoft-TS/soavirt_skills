# Skill 020: REST Client Lifecycle (Service Definition = None)

## 1) Skill Name
REST Client lifecycle in None mode (create/read/update/delete/copy + optional reorder/validation)

## 2) Objective
Manage SOAtest REST Client tests when no OpenAPI/Swagger service definition is used (or when a user only provides endpoint/method/payload details), including CRUD operations and copy.

Default JSON behavior: when a JSON payload is required, configure payload mode as `formJSON` and populate a beautified JSON literal body.

## 3) Scope
- In scope:
  - create REST Client via `POST /v6/tools/restClients`
  - update REST Client via `PUT /v6/tools/restClients?id=...`
  - read REST Client via `GET /v6/tools/restClients?id=...`
  - delete REST Client via `DELETE /v6/tools?id=...`
  - copy REST Client via `POST /v6/tools/copy`
  - reorder tool placement under suite via `PUT /v6/suites/children?id=...`
  - focused runtime validation via `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - creating a fully OpenAPI-bound REST Client from scratch through API only
  - schema-driven operation picker automation (Swagger UI-equivalent flow)

## 4) Inputs
- Required:
  - parent suite id
  - REST test name
  - HTTP method (`GET`, `POST`, `PUT`, `DELETE`, etc.)
  - endpoint template (fixed or parameterized)
- Optional:
  - expected response code(s)
  - headers (for example `Accept`, `Authorization`)
  - payload body (for `POST`/`PUT`/`PATCH`)
  - data source parameterization tokens (for example `${DB_ID}`)
  - deterministic placement index in suite
  - copy target parent/name

### 4.1) Input Gate (Required)
- Never infer or silently apply endpoint, method, headers, payload, or expected status values from prior examples/runs.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context that has been explicitly confirmed by the user.
- If create/update intent is underspecified, pause mutation and solicit missing values.

### 4.2) Conditional Required Matrix
- Always required for create:
  - parent suite id
  - REST test name
  - HTTP method
  - endpoint template
- Required for `POST`/`PUT`/`PATCH` outside scaffold mode:
  - payload content type
  - payload body source/value (literal or parameterized mapping)
- Required for negative/security-focused tests before final verification:
  - explicit expected response code strategy (`misc.validHttpResponseCodes`)

### 4.3) Missing-Field Solicitation Contract
When required fields are missing, ask for all unresolved values in one prompt:
- Method:
- Endpoint URL/template:
- Headers (optional):
- Content-Type (required for POST/PUT/PATCH outside scaffold):
- Body payload (required for POST/PUT/PATCH outside scaffold):
- Expected response codes:
Do not call create/update endpoints until required fields are provided/confirmed.

### 4.4) Scaffold-Only Create Mode
- Supported only when user explicitly requests blank/scaffold REST Client creation.
- In scaffold mode:
  - create with minimal endpoint/method shell and no assumed payload/headers,
  - mark/report configuration as incomplete immediately,
  - list remaining required fields for operational readiness.
- Do not enter scaffold mode implicitly.

## 5) Preconditions
- API reachable and authenticated.
- Target suite exists.
- If parameterized endpoint is used, referenced variables/data bank columns exist at runtime.
- If ordered placement is required, current child list can be read first.

## 6) Procedure
1. Inspect suite children and capture ordering baseline:
   - `GET /v6/suites/testSuites?id=<suite-id>` (read `relationships.childrenRel`)
2. Create REST Client in None mode:
   - enforce Input Gate and Solicitation Contract before create (unless explicit scaffold-only mode is requested).
   - `POST /v6/tools/restClients`
   - required payload parts:
     - `parent.id`
     - `name`
     - `header.method.methodType`
     - `resource.type = literalText`
     - `resource.literalText.fixed = <endpoint-template>`
3. Configure method/resource/body and response expectations:
   - `PUT /v6/tools/restClients?id=<tool-id>` with:
    - **PUT safety rule (required):** perform read-merge-write for REST Client updates.
      - first read `GET /v6/tools/restClients?id=<tool-id>`
      - merge intended edits into current config
      - submit merged PUT preserving existing `resource`, `header`, `httpOptions`, `payload`, and `misc` unless intentionally changed
      - avoid sparse/name-only PUT payloads for REST Clients
     - `header.method.methodType`
     - `resource.literalText.fixed`
  - `httpOptions.transport.http11.httpHeaders` (for example `Accept: application/json`, `Authorization: ...`)
     - header ownership rule for client tools:
       - do **not** explicitly set `Content-Type` header for REST Client requests.
       - set `payload.contentType` and allow SOAtest client tooling to emit the correct `Content-Type` automatically.
       - same rule should be applied to SOAP Client, GraphQL Client, and Messaging Client skills.
     - JSON body default (when body is needed and content type is JSON):
       - `payload.contentType = application/json`
       - `payload.inputMode = formJSON`
       - `payload.input.literal.text = <beautified JSON>`
     - non-JSON body default:
       - `payload.inputMode = literal`
     - `payload.contentType` (for example `application/json`)
     - `misc.validHttpResponseCodes.type=fixed` and `misc.validHttpResponseCodes.fixed`
   - if payload/expected-code values are orchestration-generated samples, obtain user acceptance (accept/edit) before final PUT.
4. Read back and verify the tool:
   - `GET /v6/tools/restClients?id=<tool-id>`
  - verify `resource.literalText.fixed` is non-empty (critical execution precondition)
  - for JSON payloads, verify `payload.inputMode = formJSON`
  - if beautified JSON is required in persisted `.tst` YAML (UI-friendly formatting), verify `MessagingClient_LiteralMessage: |-` in downloaded YAML and apply a no-BOM YAML upload correction if needed.
5. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied tool via `GET /v6/tools/restClients?id=<new-id>` and parent readback.
6. Optional delete flow:
  - `DELETE /v6/tools?id=<tool-id>`
  - verify deletion via descendants/children readback.
7. Reorder tool to required position (if needed):
   - build full ordered child list
   - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id` sequence
8. Execute focused validation against selected tests:
   - `POST /v6/testExecutions`
   - `general.config` must be workspace-valid (for example `soatest.user://Example Configuration`)
   - scope to target `.tst` + `soatestOptions.testNames`
9. Retrieve run evidence:
   - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
   - verify target test statuses in `xmlReport`
10. Expected-code calibration for negative/security tests (required when non-happy tests are present):
  - retrieve runtime traffic for executed tests,
  - extract observed HTTP status codes,
  - update `misc.validHttpResponseCodes.fixed` to observed behavior when initial assumptions differ,
  - run one verification execution after calibration.
  - if expected-code policy is still ambiguous after calibration, pause and request user confirmation rather than assuming.

## 7) Validation
- Expected HTTP status codes (API transport only):
  - create/update/read/reorder and execution/results calls typically return `200` on valid API requests.
  - non-`200` from the tool-under-test endpoint is diagnostic input, not automatically a test failure by itself.
- Expected response shape:
  - REST Client readback includes `header.method`, `resource`, `payload`, and `misc.validHttpResponseCodes`.
  - execution results include `summary` and `xmlReport` when requested.
- Post-condition checks:
  - tool appears under intended suite parent.
  - endpoint/method/body settings match requested configuration.
  - effective accepted status set is explicitly derived from `misc.validHttpResponseCodes`:
    - empty (`""`) => default expected `200`
    - single value => only that code
    - range => any value in range
    - comma-separated set/ranges => union of all listed expectations
  - runtime result interpretation includes chained-tool context (for example JSON Assertor/JSON Validator/Diff Tool) so secondary failures are not misclassified as primary transport failures.
  - when negative/security variants exist, expected-code values are calibrated from observed runtime statuses before final verification.

## 8) Failure Modes
- `400` invalid payload shape or field incompatibility.
- `404` invalid id or missing configuration profile during test execution.
- `405` wrong endpoint/verb combinations (for example class-specific delete endpoint mismatch).
- Sparse PUT overwrite risk: partial REST Client PUT payload (for example name-only) can clear required fields such as URL/resource and cause zero-test/no-op execution behavior.
- Copy-parent/name conflicts can cause placement ambiguity without immediate parent readback.
- runtime failures caused by missing variable bindings (for example `${DB_ID}` not produced upstream).
- request-header drift risk: explicitly setting `Content-Type` header can conflict with client-tool managed payload settings.
- Misclassification risk: treating HTTP status alone as root cause without evaluating `misc.validHttpResponseCodes`.
- Misclassification risk: treating non-`200` as failure when configured accepted codes allow it (for example `302, 500-599`).
- Cascade risk: upstream response shape/content mismatch can trigger downstream chained-tool failures (JSON Assertor/Validator/Diff) that are secondary, not primary transport issues.
- JSON payload normalization caveat:
  - REST Client API readback may compact JSON literal text even when payload mode is `formJSON`.
  - If human-readable beautified JSON must remain in `.tst`, enforce it via YAML round-trip (UTF-8 without BOM).
- Silent-default risk:
  - using example endpoint/payload/expected-code values when user/orchestration did not provide or confirm them can create incorrect tests.

## 9) Safety / Rollback
- Write skill (mutates `.tst` test structure/configuration).
- Rollback options:
  - delete new tool via `DELETE /v6/tools?id=<tool-id>`
  - restore prior suite order from saved `children` snapshot
  - reapply previous tool configuration via saved GET payload

## 10) Reuse Notes
- Applies to SOAtest: Yes (validated).
- Applies to Virtualize: not validated.
- Related endpoints:
  - `POST/PUT/GET /v6/tools/restClients`
  - `DELETE /v6/tools`
  - `POST /v6/tools/copy` (optional for cloning patterns)
  - `PUT /v6/suites/children`
  - `POST /v6/testExecutions`
  - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Composition note:
  - After REST client creation, optionally compose downstream validators/assertors based on user-requested test intent (see `skill-010-json-assertor-workflow.md` + `skill-011-xpath-over-json-query-semantics.md`).
  - For updates to existing REST Clients, load `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` and use GET -> mutate -> PUT.

### JSON Tooling Reminder
- When this REST Client is chained to JSON tools (for example JSON Assertor, JSON Data Bank), selector-expression fields use XPath-over-JSON semantics.
- Always load `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md` for JSON selector authoring; do not use JSONPath in XPath-expected fields.

## 11) Current Validation Status (2026-03-03)
Validated in workspace `soavirt_skills` with root test `/accounts/{accountId} - GET` configured in None-mode behavior and executed after root DB tool.

Evidence artifacts:
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/refine_put_root_accounts_from_childcopy_response.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/refine_root_children_reorder_response.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/refine_execution_results_1283881965.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/Basic_Test_Construction_Learning_refined_final_dump.txt`

Additional validated replacement scenario (same day):
- Replaced the 3rd root node REST Client (after DB tool) with a new fixed-endpoint GET test for:
  - `GET http://localhost:8090/parabank/services/bank/accounts/12345`
  - `Accept: application/json`
- Preserved suite position by rebuilding full root child order and applying `PUT /v6/suites/children`.

Evidence artifacts:
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_test_root_suite_before.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_test_create_new_restclient_response.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_test_reorder_payload.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_test_reorder_response.json`
- `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_test_root_suite_after.json`

Additional validated JSON payload scenario (2026-03-04):
- Created and appended REST Client:
  - `POST /parabank/services/bank/billpay - Sample Payee`
  - endpoint with required query params: `accountId`, `amount`
  - JSON `Payee` payload configured with `payload.inputMode=formJSON`
- Verified persisted YAML mode:
  - `mode: Form JSON`
- Verified beautified JSON persistence via YAML upload:
  - `MessagingClient_LiteralMessage: |-` block style

Evidence artifacts:
- `work/runs/2026-03-04/billpay-rest-client/14_billpay_restclient_summary.json`
- `work/runs/2026-03-04/billpay-rest-client/25_billpay_block_final.txt`
- `work/runs/2026-03-04/billpay-rest-client/26_billpay_api_summary_final.json`
