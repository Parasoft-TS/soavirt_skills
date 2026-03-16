# Skill 020: REST Client Lifecycle (Service Definition = None)

## 1) Skill Name
REST Client lifecycle in None mode (create/read/update/delete/copy + optional reorder/validation)

## 2) Objective
Manage SOAtest REST Client tests when no OpenAPI/Swagger service definition is used (or when a user only provides endpoint/method/payload details), including create/read/update work plus subordinate copy/delete steps when they are part of broader REST Client lifecycle activity. This card also owns caller-directed in-place normalization of already-generated unconstrained REST Clients when a broader workflow needs to replace a concrete host with a confirmed variable template such as `${BASEURL}` plus relative path/query.
Pure rename/copy/delete/enable-disable requests on an existing REST Client should route to the centralized operation-centric owners in `docs/skills/structure/skill-068-rename-object.md`, `docs/skills/structure/skill-070-copy-tool.md`, `docs/skills/structure/skill-072-delete-tool.md`, and `docs/skills/structure/skill-074-set-disabled-state.md`.

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
  - editing existing service-definition-backed constrained REST Clients via YAML fallback (owned by `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`)

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- Additive:
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`

## 4) Inputs
- Required:
  - parent suite id
  - HTTP method (`GET`, `POST`, `PUT`, `DELETE`, etc.)
  - endpoint template (fixed or parameterized)
- Optional:
  - REST test name (optional - derive using `{endpoint} - {method}` when not provided, for example `/billpay - POST`; follow `docs/skills/cross-cutting/skill-013-test-naming-policy.md`)
  - expected response code(s) (optional - default to `200` unless user specifies otherwise)
  - headers (for example `Accept` or endpoint-specific custom headers)
  - auth configuration when required by the target service (validated runtime-supported path: Basic auth through the native REST Client auth subtree)
  - payload body (for `POST`/`PUT`/`PATCH`; explicit empty body is valid)
  - inferred payload content type when body is provided (if body is intentionally empty, content type may be omitted unless user explicitly provides one)
  - data source parameterization tokens (for example `${DB_ID}`)
  - deterministic placement index in suite
  - copy target parent/name
### 4.0) Create Mode Selection (Required)
- Two create modes are supported:
  - `configured` (default): create a ready-to-run REST Client with required method/endpoint/body/expectation inputs.
  - `scaffold-only`: create an empty/minimal REST Client shell and defer operational fields.
- If the user does not explicitly request scaffold mode, proceed as `configured`.

### 4.1) Input Gate (Required)
- Never infer or silently apply endpoint, method, headers, payload, or expected status values from prior examples/runs.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context or active project record values that have been explicitly confirmed by the user for this branch.
- If create/update intent is underspecified, pause mutation and solicit missing values.
- In project-aware branches, consult active project facts/references/environment files for candidate base URLs, auth/header variable tokens, stored secret-reference paths, and known request-value anchors before asking the user to restate them, but do not silently treat those candidates as approved.
- Do not switch to scaffold mode implicitly.

### 4.2) Conditional Required Matrix
- Create mode:
  - `configured` by default
  - `scaffold-only` only when user explicitly requests it
- Always required for create:
  - parent suite id
  - HTTP method
  - endpoint template
- Required for `POST`/`PUT`/`PATCH` outside scaffold mode:
  - payload body source/value (literal or parameterized mapping); explicit empty-body intent is valid
  - `payload.contentType` should be inferred from provided body when possible; if body is intentionally empty, content type may be omitted unless user explicitly provides one
- Required for negative/security-focused tests before final verification:
  - explicit expected response code strategy (`misc.validHttpResponseCodes`)

### 4.3) Missing-Field Solicitation Contract
When required fields are missing for the selected create mode, ask for all unresolved values in one prompt:
- Create mode (`configured` or `scaffold-only`; default `configured`):
- Method:
- Endpoint URL/template:
- REST Client name (optional - I'll infer using `{endpoint} - {method}` unless you want something specific):
- Headers (optional):
- Body payload (required for POST/PUT/PATCH outside scaffold; explicit empty body is allowed):
- Expected response codes (optional - I'll default to `200` unless you tell me otherwise):
Do not call create/update endpoints until required fields for the selected mode are provided/confirmed.

### 4.4) Scaffold-Only Create Mode
- Supported only when user explicitly requests blank/scaffold REST Client creation.
- In scaffold mode:
  - create with minimal endpoint/method shell and no assumed payload/headers,
  - mark/report configuration as incomplete immediately,
  - list remaining required fields for operational readiness.
- Do not enter scaffold mode implicitly.

### 4.5) REST Client Auth Ownership
- When auth is required and the REST Client auth model supports the requested scheme, configure auth through `httpOptions.transport.<http10|http11>.security.httpAuthentication.performAuthentication` instead of injecting a literal `Authorization` header.
- Validated runtime-supported auth path in this workspace: `basic`.
- Schema-visible but unvalidated in this workspace: `ntlm`, `kerberos`, `digest`.
- Basic auth wrapper types:
  - username uses the `complexValueFP` shape (for example `{"type":"fixed","fixed":"<username>"}`)
  - password uses the `complexValueMP` shape (for example `{"type":"masked","masked":"<password>"}`)
- Literal `Authorization` headers are reserved for unsupported or intentionally custom schemes (for example future OAuth2 work) and should not replace the validated Basic-auth path.
- Approved secret-reference sources (for example a gitignored path resolved from project context) are valid auth input provenance for supported Basic-auth write branches.
- When such a source is available and the owning workflow intends auth configuration, resolve the credentials and materialize them directly into the native auth subtree.
- Do not reroute into referenced-environment or other concealment branches solely to avoid persisting credentials in the `.tst`; protecting the generated asset before source-control commit remains the user's responsibility unless the user explicitly asks for another safeguard.
- Stop rather than guess when the active transport branch cannot be resolved to `http10` or `http11`.

### 4.6) Request-Payload Parameterization Rule
- When a REST request body needs datasource-backed parameterization, target the primitive leaf value that should vary, not the enclosing object or array subtree.
- Validated pattern: place `${TOKEN}` directly at the intended primitive leaf inside `payload.input.literal.text` / beautified JSON.
- In SOAtest/Virtualize terminology, the referenced "data source column" may come from either a true datasource-backed column or a tool-produced custom column such as Data Bank or Data Generator output.
- API readback may preserve the literal `${TOKEN}` body text, while the persisted local `.tst` may normalize that same leaf into structured `formJson` datasource wiring such as `mode: 3` with `columnName: TOKEN`.
- Keep the surrounding request-body shape intact around the parameterized leaf.
- Do not generalize this evidence to complex-object or whole-subtree substitution; primitive leaf values are the validated boundary for this card today.

## 5) Preconditions
- API reachable and authenticated.
- Target suite exists.
- If parameterized endpoint is used, referenced variables/data bank columns exist at runtime.
- If ordered placement is required, current child list can be read first.

## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile E for tool mutation steps (`POST`/`PUT`/`DELETE` and suite reorder),
   - use Skill 050 Profile D for execution/traffic-observation steps.
1. Inspect suite children and capture ordering baseline:
   - `GET /v6/suites/testSuites?id=<suite-id>` (read `relationships.childrenRel`)
2. Create REST Client in None mode:
   - enforce Input Gate and Solicitation Contract before create.
   - branch by create mode:
     - configured (default):
       - require all configured-mode fields per Section 4.2 before create.
     - scaffold-only (explicit user request only):
       - allow minimal shell creation with deferred payload/headers/expected-codes.
   - `POST /v6/tools/restClients`
   - required payload parts:
     - `parent.id`
     - `name`
     - `header.method.methodType`
     - `resource.type = literalText`
     - `resource.literalText.fixed = <endpoint-template>`
2.1 Resolve REST Client identity before create (idempotent create rule):
   - derive deterministic target id as `<suite-id>/<rest-test-name>`,
   - if that id exists, update it in place,
   - create via `POST /v6/tools/restClients` only when target REST Client is absent,
   - do not create another REST Client with the same intent/name in the same parent.
3. Configure method/resource/body and response expectations:
   - `PUT /v6/tools/restClients?id=<tool-id>` with:
     - **PUT safety rule (required):** perform read-merge-write for REST Client updates.
      - first read `GET /v6/tools/restClients?id=<tool-id>`
      - merge intended edits into current config
      - submit merged PUT preserving existing `resource`, `header`, `httpOptions`, `payload`, and `misc` unless intentionally changed
      - avoid sparse/name-only PUT payloads for REST Clients
     - key edited fields typically include `header.method.methodType` and `resource.literalText.fixed`
     - caller-owned generated-template normalization branch:
       - when the caller has already established a local base variable contract (for example `BASEURL`) and the current `resource.literalText.fixed` is a concrete URL rooted at that confirmed base, rewrite it during the same read-merge-write PUT to `${BASEURL}` plus the preserved relative path/query
       - preserve already-parameterized, relative, or non-matching endpoint templates as-is
       - stop rather than guess when the current URL cannot be safely decomposed into confirmed base plus relative remainder
   - `httpOptions.transport.http11.httpHeaders` (for example `Accept: application/json` or endpoint-specific custom headers)
     - header ownership rule for client tools:
       - do **not** explicitly set `Content-Type` header for REST Client requests.
       - set `payload.contentType` and allow SOAtest client tooling to emit the correct `Content-Type` automatically.
       - for broader client-header ownership policy and cross-client consistency, follow `docs/skills/cross-cutting/skill-032-client-header-ownership.md`.
     - auth ownership rule for REST Clients:
       - when the requested auth scheme is supported by the native REST Client auth model, prefer `httpOptions.transport.<http10|http11>.security.httpAuthentication.performAuthentication` over literal `Authorization` header injection.
       - resolve the existing transport branch from GET readback first; do not invent or silently swap between `http10` and `http11`.
       - validated runtime-supported auth path in this workspace: `basic`.
       - schema-visible but unvalidated: `ntlm`, `kerberos`, `digest`.
       - Basic auth update contract:
         - set `enabled = true`
         - set `value.useGlobal = false` unless the user explicitly wants global auth
         - set `value.authenticationType.type = basic`
         - set `value.authenticationType.basic.username` with a `complexValueFP` wrapper (for example `{"type":"fixed","fixed":"<username>"}`)
         - set `value.authenticationType.basic.password` with a `complexValueMP` wrapper (for example `{"type":"masked","masked":"<password>"}`)
         - if auth input came from an approved secret reference, resolve the plaintext credential values before the PUT and write them directly into the native auth subtree
         - do not divert to referenced-environment or concealment workarounds solely to avoid persisting those values in the `.tst`
       - preserve unrelated `httpOptions` fields through read-merge-write and verify the auth subtree with a final GET readback.
       - reserve literal `Authorization` headers for unsupported or intentionally custom schemes (for example future OAuth2 work).
       - stop rather than guess when the transport branch cannot be resolved, the requested scheme is outside the validated Basic path, or readback does not preserve the expected auth subtree.
     - JSON body default (when body is needed and content type is JSON):
       - `payload.contentType = application/json`
       - `payload.inputMode = formJSON`
       - `payload.input.literal.text = <beautified JSON>`
     - non-JSON body default:
       - `payload.inputMode = literal`
     - set `payload.contentType` explicitly for payload-bearing methods (for example `application/json`)
     - `misc.validHttpResponseCodes.type=fixed` and `misc.validHttpResponseCodes.fixed`
   - if payload/expected-code values are orchestration-generated samples, obtain user acceptance (accept/edit) before final PUT.
4. Read back and verify the tool:
   - `GET /v6/tools/restClients?id=<tool-id>`
   - verify `resource.literalText.fixed` is non-empty (critical execution precondition)
   - for JSON payloads, verify `payload.inputMode = formJSON`
   - if Basic auth was configured through the native auth model, verify the selected transport branch preserves `security.httpAuthentication.performAuthentication.enabled = true`, `authenticationType.type = basic`, and the expected username/password wrapper shapes
   - if beautified JSON is required in persisted `.tst` YAML (UI-friendly formatting), verify `MessagingClient_LiteralMessage: |-` in downloaded YAML and apply a no-BOM YAML upload correction if needed.
5. Optional copy flow (substep only):
   - for pure copy-only intent, prefer `docs/skills/structure/skill-070-copy-tool.md`
   - keep this branch only when copy is subordinate to broader REST Client lifecycle/configuration work
   - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
   - verify copied tool via `GET /v6/tools/restClients?id=<new-id>` and parent readback.
6. Optional delete flow (substep only):
   - for pure delete-only intent, prefer `docs/skills/structure/skill-072-delete-tool.md`
   - keep this branch only when delete is subordinate to broader REST Client lifecycle/configuration work
   - `DELETE /v6/tools?id=<tool-id>`
   - verify deletion via descendants/children readback.
7. Reorder tool to required position (if needed):
   - build full ordered child list
   - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id` sequence
8. Execute focused validation against selected tests by following the Skill 012 execution-triad contract.
   - use a workspace-valid `general.config` (for example `soatest.user://Example Configuration`)
   - scope execution to the target `.tst` plus `soatestOptions.testNames`
9. Use the Skill 012 results/readback step to verify target test statuses in `xmlReport`.
10. Expected-code calibration for negative/security tests (required when non-happy tests are present):
   - retrieve runtime traffic for executed tests and use only test-execution traffic evidence for calibration (do not calibrate from direct endpoint calls outside the test run),
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
  - when Basic auth is configured through the native auth model, readback preserves the selected transport branch, `performAuthentication.enabled = true`, and the expected `complexValueFP`/`complexValueMP` wrapper shapes for username/password.
  - if create mode was scaffold-only, result is explicitly reported as incomplete with remaining fields listed.
  - effective accepted status set is explicitly derived from `misc.validHttpResponseCodes`:
    - empty (`""`) => default expected `200`
    - single value => only that code
    - range => any value in range
    - comma-separated set/ranges => union of all listed expectations
  - runtime result interpretation includes chained-tool context (for example JSON Assertor/JSON Validator/Diff Tool) so secondary failures are not misclassified as primary transport failures.
  - when negative/security variants exist, expected-code values are calibrated from observed runtime statuses before final verification.
  - when Basic auth was updated, readback confirmation of the selected transport branch and auth subtree is sufficient configuration proof unless the caller separately asked for runtime verification.
  - when caller-owned template normalization rewrote a generated endpoint to `${BASEURL}` plus relative path/query, readback preserves that parameterized endpoint template exactly.

## 8) Failure Modes
- `400` invalid payload shape or field incompatibility.
- `404` invalid id or missing configuration profile during test execution.
- `405` wrong endpoint/verb combinations (for example class-specific delete endpoint mismatch).
- Sparse PUT overwrite risk: partial REST Client PUT payload (for example name-only) can clear required fields such as URL/resource and cause zero-test/no-op execution behavior.
- Copy-parent/name conflicts can cause placement ambiguity without immediate parent readback.
- runtime failures caused by missing variable bindings (for example `${DB_ID}` not produced upstream).
- request-header drift risk: explicitly setting `Content-Type` header can conflict with client-tool managed payload settings.
- auth-ownership drift risk: injecting a literal `Authorization` header for a supported Basic-auth REST Client can diverge from the native auth model and obscure the real persisted auth state.
- transport-branch ambiguity risk: applying auth to a guessed `http10` or `http11` branch instead of the branch proven by GET readback can create no-op or misleading updates.
- unsupported-auth drift risk: `ntlm`, `kerberos`, or `digest` may be schema-visible but are not validated runtime-supported authoring paths in this workspace.
- auth-materialization drift risk: a supported Basic-auth write branch detours into referenced-environment or other concealment workarounds even though an approved secret reference could have been materialized directly into the native auth subtree.
- over-validation risk: treating auth wiring as incomplete without a separate execution run can add unnecessary work when the user's goal is configuration authoring rather than runtime proof.
- Misclassification risk: treating HTTP status alone (or assuming non-`200` is failure) without evaluating `misc.validHttpResponseCodes` (for example configured `302, 500-599` acceptance).
- Cascade risk: upstream response shape/content mismatch can trigger downstream chained-tool failures (JSON Assertor/Validator/Diff) that are secondary, not primary transport issues.
- JSON payload normalization caveat:
  - REST Client API readback may compact JSON literal text even when payload mode is `formJSON`.
  - If human-readable beautified JSON must remain in `.tst`, enforce it via YAML round-trip (UTF-8 without BOM).
- Silent-default risk:
  - using example endpoint/payload/expected-code values when user/orchestration did not provide or confirm them can create incorrect tests.
- Intent-mismatch risk:
  - creating scaffold when user intended configured (or vice versa) causes avoidable rework.
- Template-normalization drift risk:
  - a caller-owned rewrite converts a non-matching or already-parameterized generated endpoint into `${BASEURL}` even though safe decomposition was not proven.

## 9) Safety / Rollback
- Write skill (mutates `.tst` test structure/configuration).
- Rollback options:
  - delete new tool via `DELETE /v6/tools?id=<tool-id>`
  - restore prior suite order from saved `children` snapshot
  - reapply previous tool configuration via saved GET payload

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
- Related endpoints:
  - `POST/PUT/GET /v6/tools/restClients`
  - `DELETE /v6/tools`
  - `POST /v6/tools/copy` (optional for cloning patterns)
  - `PUT /v6/suites/children`
  - `POST /v6/testExecutions`
  - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Composition note:
  - After REST client creation, if the user's intent is broader validation enrichment on existing/new tests, route to `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md` rather than composing validator/assertor/diff leaves ad hoc.
  - Use direct validator/assertor/diff leaves only when the routing is already explicit and the downstream tool choice is intentionally narrowed.
  - For same-operation edits on existing constrained REST Clients (`resource.type=swagger` / constrained YAML model), route to `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md` rather than treating Skill 020 as the mutation owner.
  - For pure rename/copy/delete/enable-disable prompts on an existing REST Client, prefer the centralized operation-centric owners; keep Skill 020 for broader REST Client lifecycle/configuration work.
  - Broad-authoring callers such as Skill 065 may use this card immediately after Skill 022 generation to normalize generated template-suite REST Clients from concrete hosts to `${BASEURL}` plus relative path/query once Skill 064 has inserted the local environment variable.
