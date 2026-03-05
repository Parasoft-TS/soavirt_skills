# Skill 029: JSON Validator Workflow (Cross-Tool)

## 1) Skill Name
Add and validate JSON Validator on JSON-producing tool output

## 2) Objective
Create a reusable workflow for chaining a JSON Validator to JSON output, defaulting to schema validation with explicit schema/service-definition configuration and explicit message mapping, and validating runtime payload structure/content against validator configuration.

When adding a JSON Validator to tools such as REST Clients, default behavior is schema validation (`validateAgainstSchema`) with service-definition location configured. If service-definition/schema inputs cannot be inferred with high confidence, prompt the user to provide them before configuration.

## 3) Scope
- In scope:
  - create JSON Validator via `POST /v6/tools/jsonValidators`
  - configure JSON Validator via `PUT /v6/tools/jsonValidators?id=...`
  - readback via `GET /v6/tools/jsonValidators?id=...`
  - run focused execution and validate runtime behavior via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - auto-remediation of payload/schema issues
  - domain-specific JSON schema design guidance

## 4) Inputs
- Required:
  - target output-provider parent id that emits JSON
  - JSON Validator name
  - OpenAPI/Swagger service-definition location (URL or resolvable file/id) for schema validation configuration
- Optional:
  - explicit validator configuration settings (validation type, schema/service-definition location)
  - schema/service-definition hints from prior user prompts
  - schema/service-definition hints from environment variables
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits JSON payloads.
- Parent id resolves to an output-provider location that accepts chained tools.
- Target producer is an API client tool response output (REST Client today; SOAP/Messaging client outputs when those skills are available).
- Runtime response media type for target output is JSON (confirmed from baseline run evidence).
- JSON selector/query rule must be loaded when selector fields are used:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - use XPath-style selectors for JSON fields, not JSONPath.

## 6) Procedure
1. Construct the output-provider parent path for the target producer tool:
  - REST Client: `<rest-client-id>/Response Traffic`
  - API client tools only: use their semantic JSON response output-provider anchor.
  - Do not target DB Tool outputs for JSON Validator schema-validation workflows.
  - Do NOT pass the producer tool id directly as `parent.id`; tool creation requires `parent.id` of type `outputProvider` or `suite`. Passing a REST Client id (type `REST Client`) returns `400`.
  - Do NOT attempt to discover output provider ids via `/children` or `/descendants/assets`; empty output providers are not returned. Construct the path directly.
  - See `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` Section 5 for canonical patterns.
1.1 Run baseline execution and confirm observed response media type is JSON before creating/updating JSON Validator.
  - if payload is XML, route to Skill 030.
  - if payload is plain text, route to Skill 031 (text mode).
2. Resolve validator identity before writes (idempotent upsert rule):
  - derive deterministic target id as `<parent-id>/<validator-name>`,
  - if that id exists, update it in place,
  - create via `POST /v6/tools/jsonValidators` only when validator is truly absent.
  - do not create another validator with the same intent/name in the same parent.
3. Build candidate schema/service-definition inputs using this precedence:
  - explicit values provided in the current user request,
  - schema/service-definition values stated earlier in the current session,
  - relevant environment variables already configured for this test context,
  - inferable service-definition hints from existing tools and their children in the target `.tst` (for example REST Client lineage, generated suite/source naming, and validator-capable sibling configuration).
4. Apply confidence gate before configuring schema validation:
  - proceed only when one candidate is unambiguous and format-compatible with the JSON payload,
  - if candidates conflict, are missing, or are weakly inferred, ask a targeted user question and wait for confirmation.
  - do not generate synthetic/placeholder schemas to bypass missing schema/service-definition input.
  - do not create/configure JSON Validator in `checkWellFormednessOnly` mode because definition inputs are unknown.
4.1 OpenAPI-known strict rule:
  - when the orchestration context provides a known OpenAPI/Swagger location for the target operation, do not use `checkWellFormednessOnly`,
  - configure `validationType=validateAgainstSchema` with explicit OpenAPI message mapping (`autoDetectMessage=false`, direction/path/method/responseCode),
  - if mapping cannot be resolved for this endpoint, fail/skip this endpoint validator explicitly and continue other endpoints; do not downgrade unrelated JSON validators.
4.2 Missing-definition prompt rule (required):
  - when service-definition/schema inputs cannot be inferred from user input, session context, environment variables, or tool/tree inspection with high confidence, prompt the user to provide schema/service-definition location.
  - treat this as a blocking gate for schema-mode configuration; wait for user input before completing validator configuration.
  - do not default to `checkWellFormednessOnly` only because definition inputs are missing.
  - only default to `checkWellFormednessOnly` if the user agrees to that
5. Create JSON Validator when absent:
   - `POST /v6/tools/jsonValidators` with `parent.id` and `name`.
6. Configure validator settings:
   - `PUT /v6/tools/jsonValidators?id=<validator-id>`.
  - default to `toolSettings.validationType=validateAgainstSchema`.
  - set `toolSettings.schemaValidationSettings` from explicit or high-confidence inferred inputs.
  - when using OpenAPI service definitions, default to explicit message mapping instead of auto-detect:
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.autoDetectMessage=false`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.direction=response`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.path=<resolved response path>`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.method=<resolved HTTP method>`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.responseCode=<expected code, default 200>`
  - resolve message path/method from the chained producer context and use runtime response context to confirm.
  - for path-parameter operations, keep service-definition template placeholders in message mapping (for example `/customers/{customerId}`), not concretized runtime values (for example `/customers/12212`).
  - use `checkWellFormednessOnly` only when user explicitly requests it.
  - do not use `checkWellFormednessOnly` when rule 4.1 conditions are satisfied.
7. If selector-expression fields are present, apply Skill 011 semantics (XPath over JSON).
8. Read back tool and verify configuration persisted.
9. Execute focused run and inspect runtime results.

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback includes persisted validator settings.
- Repeat executions/update passes do not create additional validators for the same parent/name intent.
- Readback confirms default `validationType` is `validateAgainstSchema` unless user explicitly selected `checkWellFormednessOnly`.
- For OpenAPI-backed setup, readback confirms explicit message mapping fields are populated (direction/path/method/responseCode) with `autoDetectMessage=false`.
- Inferred schema/service-definition values are only applied when confidence gate conditions are met.
- When definition inputs are not inferable with high confidence, workflow blocks on user prompt rather than downgrading validation mode.
- When rule 4.1 applies, readback confirms `validationType=validateAgainstSchema` and explicit OpenAPI mapping fields are populated.
- Runtime result shows validator pass/fail aligned to payload validity and configuration.

## 8) Failure Modes
- `400` invalid payload shape or unsupported validator settings.
- `400` when `parent.id` resolves to type `REST Client` instead of `outputProvider`; use `<rest-client-id>/Response Traffic` path.
- `404` invalid tool id or parent id.
- Standalone validator with no input: if created under a suite (not under an output provider), the validator has no chained input to validate and fails with `Invalid JSON: unable to find a JSON object...`.
- Selector mismatch when JSONPath is used where XPath-over-JSON is expected.
- Media-type mismatch when JSON Validator is attached to XML/plain-text traffic.
- Scope-violation risk when JSON Validator is attached to DB Tool outputs (JSON Validator workflow is API-client response-output only).
- Missing schema/service-definition configuration when default schema-validation mode is expected.
- User prompt required when schema/service-definition cannot be inferred may pause orchestration until input is provided (intentional blocking behavior).
- Incorrect downgrade risk: using well-formedness mode despite known OpenAPI location and JSON response.
- OpenAPI auto-detect picks the wrong message/schema node for the runtime response when auto-detect is explicitly enabled as an override.
- Incomplete explicit OpenAPI message mapping (for example missing `direction`) causes payload validation setup errors.
- Wrong schema/service-definition inferred from ambiguous context.
- Synthetic schema generation used instead of user-provided/verified schema location.
- Input-binding mismatch if chained to diagnostics-only output instead of semantic JSON output.

## 9) Safety / Rollback
- Write skill (adds/modifies validator tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<validator-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Applies to SOAtest: expected; runtime validation pending first dedicated run.
- Applies to Virtualize: not yet validated.
- Intended producer class: API client tool response outputs (REST Client now; SOAP/Messaging when corresponding skills are available).
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`

## 11) Initial Validation Plan
1. Create under REST Client JSON response output.
2. Resolve schema/service-definition using precedence + confidence gate (ask user when uncertain).
3. Configure baseline validation settings with default `validateAgainstSchema`.
4. Execute focused run and capture pass/fail behavior.
