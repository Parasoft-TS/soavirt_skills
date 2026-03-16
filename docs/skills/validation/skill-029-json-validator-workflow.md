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
  - run focused execution and validate runtime behavior by following the Skill 012 execution-triad contract
- Out of scope:
  - auto-remediation of payload/schema issues
  - domain-specific JSON schema design guidance

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- When called from Skill 067, this card may rely on upstream exploration-backed payload classification, target parent selection, approved validator-family intent, and candidate schema-source basis, while still owning local validator create/update/readback/verification mechanics.

## 4) Inputs
- Required:
  - target output-provider parent id that emits JSON
  - JSON Validator name
  - OpenAPI/Swagger service-definition location (URL or resolvable file/id) for schema validation configuration
- Optional:
  - explicit validator configuration settings (validation type, schema/service-definition location)
  - schema/service-definition hints from prior user prompts
  - schema/service-definition hints from environment variables
  - caller-supplied operation identity or validator message-mapping basis
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits JSON payloads.
- Parent id resolves to an output-provider location that accepts chained tools.
- Target producer is an API client tool response output (REST Client today; SOAP/Messaging client outputs when those skills are available).
- In the stable lane, runtime response media type for target output is JSON and must be confirmed from baseline run evidence before family selection. When this card is called from Skill 067 in the experimental lane, upstream exploration-backed payload classification, approved validator family, target parent selection, and candidate schema-source basis may satisfy that family-selection gate before attachment, but post-attachment focused verification remains required.
- JSON selector/query rule must be loaded when selector fields are used:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - use XPath-style selectors for JSON fields, not JSONPath.

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - This workflow remains API-client response-output scoped; do not target DB Tool outputs for JSON Validator schema-validation workflows.
  - Construct `parent.id` from Skill 018 mapping.
  - Do NOT pass the producer tool id directly as `parent.id`; tool creation requires `parent.id` of type `outputProvider` or `suite`. Passing a REST Client id (type `REST Client`) returns `400`.
  - Do NOT attempt to discover output provider ids via `/children` or `/descendants/assets`; empty output providers are not returned.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating/configuring JSON Validator.
  - do not guess or locally invent parent-path mappings.
1.2 Fail-closed media-type gate:
  - in the stable lane, run baseline execution and inspect the observed semantic response payload/body first; use response headers such as `Content-Type` only as supporting evidence.
  - when this card is called from Skill 067, accept upstream exploration-backed payload classification, approved validator-family intent, target parent selection, and candidate schema-source basis as authoritative for family-selection purposes instead of rerunning baseline work only to rediscover them.
  - if observed or upstream-authoritative payload is JSON, continue JSON Validator flow.
  - if observed or upstream-authoritative payload is XML, route to Skill 030 instead of creating/updating JSON Validator.
  - if observed or upstream-authoritative payload is plain text, route to Skill 031 in text mode instead of creating/updating JSON Validator.
  - do not select JSON Validator only because the producer is a REST Client or other API client tool.
2. Resolve validator identity before writes (idempotent upsert rule):
  - derive deterministic target id as `<parent-id>/<validator-name>`,
  - if that id exists, update it in place,
  - create via `POST /v6/tools/jsonValidators` only when validator is truly absent.
  - do not create another validator with the same intent/name in the same parent.
2.1 Approved-plan preservation rule:
  - if the caller/orchestration already approved JSON Validator for this target, treat validator coverage as a required deliverable for that target.
  - if JSON Validator creation or configuration cannot proceed, mark the target `blocked` or `partial` and return for a new user decision rather than silently continuing with only Diff/Assertor coverage.
3. Build candidate schema/service-definition inputs using this precedence:
  - explicit values provided in the current user request,
  - schema/service-definition values stated earlier in the current session,
  - relevant environment variables already configured for this test context,
  - inferable service-definition hints from existing tools and their children in the target `.tst` (for example REST Client lineage, generated suite/source naming, and validator-capable sibling configuration).
3.1. Preserve candidate provenance while collecting inputs:
  - if a candidate comes from an environment variable, retain both the variable form and the resolved value,
  - if a candidate is inferred from tool/tree context, record the exact source that made it plausible.
4. Apply confidence + confirmation gate before configuring schema validation:
  - proceed only when one candidate is unambiguous, format-compatible with the JSON payload, and explicitly confirmed by the user unless the user already provided that exact source in the current request,
  - if candidates conflict, are missing, weakly inferred, or only environment/tool-context derived, ask a targeted user question and wait for confirmation before use,
  - when confirming an environment-variable-backed source, echo both the variable form and the resolved value.
  - do not generate synthetic/placeholder schemas to bypass missing schema/service-definition input.
  - do not create/configure JSON Validator in `checkWellFormednessOnly` mode because definition inputs are unknown.
4.1 OpenAPI-known strict rule:
  - when the orchestration context provides a known OpenAPI/Swagger location for the target operation, do not use `checkWellFormednessOnly`,
  - configure `validationType=validateAgainstSchema` with explicit OpenAPI message mapping (`autoDetectMessage=false`, direction/path/method/responseCode),
  - if mapping cannot be resolved for this endpoint, fail/skip this endpoint validator explicitly and continue other endpoints; do not downgrade unrelated JSON validators.
4.2 Missing-definition prompt rule (required):
  - when service-definition/schema inputs cannot be inferred from user input, session context, environment variables, or tool/tree inspection with high confidence, prompt the user to provide schema/service-definition location.
  - treat this as a blocking gate for schema-mode configuration; wait for user input before completing validator configuration.
  - even high-confidence environment-variable-backed or tool/tree-inferred candidates still require explicit user confirmation before use.
  - do not default to `checkWellFormednessOnly` only because definition inputs are missing.
  - only default to `checkWellFormednessOnly` if the user agrees to that
5. Create JSON Validator when absent:
   - `POST /v6/tools/jsonValidators` with `parent.id` and `name`.
6. Configure validator settings:
   - `PUT /v6/tools/jsonValidators?id=<validator-id>`.
  - default to `toolSettings.validationType=validateAgainstSchema`.
  - set `toolSettings.schemaValidationSettings` from explicit or explicitly confirmed inferred inputs.
  - when using OpenAPI service definitions, default to explicit message mapping instead of auto-detect:
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.autoDetectMessage=false`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.direction=response`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.path=<resolved response path>`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.method=<resolved HTTP method>`
    - `toolSettings.schemaValidationSettings.definitionSettings.openapi.message.responseCode=<expected code, default 200>`
  - derive message mapping in this order:
    1. exact authored producer operation identity or caller-supplied message-mapping basis
    2. confirmed OpenAPI path template and method for that exact operation
    3. expected response-code basis from the authored test intent or approved calibration context
  - treat observed concrete runtime URLs as confirmation evidence only; do not invent message paths from paraphrased resource names, response semantics, or hand-written approximations.
  - for path-parameter operations, keep service-definition template placeholders in message mapping (for example `/customers/{customerId}`), not concretized runtime values (for example `/customers/12212`).
  - if the exact operation identity cannot be reconciled to one confirmed OpenAPI template path/method, stop and return blocked/partial rather than guessing.
  - use `checkWellFormednessOnly` only when user explicitly requests it.
  - do not use `checkWellFormednessOnly` when rule 4.1 conditions are satisfied.
7. If selector-expression fields are present, apply Skill 011 semantics (XPath over JSON).
8. Read back tool and verify configuration persisted.
9. Execute focused run by following the Skill 012 execution-triad contract and inspect runtime results.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md` (Decision Rule).
- Construct JSON Validator payloads from REST API schema + skill semantics.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Before using it, follow:
- Section 5) Preconditions
- Section 6) Procedure

Rules:
- Replace all `{{...}}` placeholders using runtime evidence + user intent.
- Do not reuse sample ids/names/paths/methods/response codes unless explicitly requested.
- Keep OpenAPI mapping explicit (`autoDetectMessage=false`) when schema mode is used.
- For updates, use `PUT /v6/tools/jsonValidators?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same `toolSettings` shape, no `parent` field).

```json
{
  "parent": {
    "id": "{{PARENT_OUTPUT_PROVIDER_ID}}"
  },
  "name": "{{JSON_VALIDATOR_NAME}}",
  "toolSettings": {
    "validationType": "validateAgainstSchema",
    "schemaValidationSettings": {
      "definitionSettings": {
        "openapi": {
          "autoDetectMessage": false,
          "message": {
            "direction": "response",
            "path": "{{OPENAPI_PATH_TEMPLATE}}",
            "method": "{{HTTP_METHOD}}",
            "responseCode": "{{RESPONSE_CODE}}"
          }
        }
      },
      "definitionLocation": {
        "url": "{{OPENAPI_URL_OR_LOCATION}}"
      }
    }
  }
}
```

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback includes persisted validator settings.
- Repeat executions/update passes do not create additional validators for the same parent/name intent.
- Readback confirms default `validationType` is `validateAgainstSchema` unless user explicitly selected `checkWellFormednessOnly`.
- For OpenAPI-backed setup, readback confirms explicit message mapping fields are populated (direction/path/method/responseCode) with `autoDetectMessage=false`.
- Explicit OpenAPI message mapping is derived from one confirmed operation template rather than from concretized runtime URLs or semantic guesswork.
- Inferred schema/service-definition values are only applied when confidence gate conditions are met and the user explicitly confirms the candidate source.
- When definition inputs are not inferable with high confidence, workflow blocks on user prompt rather than downgrading validation mode.
- Environment-variable-backed candidates are echoed using both variable form and resolved value before they are used.
- When rule 4.1 applies, readback confirms `validationType=validateAgainstSchema` and explicit OpenAPI mapping fields are populated.
- Runtime result shows validator pass/fail aligned to payload validity and configuration.
- When JSON Validator coverage was approved by the caller/orchestration, it is either delivered or explicitly reported as blocked/partial; it is not silently omitted.

## 8) Failure Modes
- `400` invalid payload shape or unsupported validator settings.
- `400` when `parent.id` resolves to type `REST Client` instead of `outputProvider`; use `<rest-client-id>/Response Traffic` path.
- `404` invalid tool id or parent id.
- Standalone validator with no input: if created under a suite (not under an output provider), the validator has no chained input to validate and fails with `Invalid JSON: unable to find a JSON object...`.
- Selector mismatch when JSONPath is used where XPath-over-JSON is expected.
- Media-type mismatch when JSON Validator is attached to XML/plain-text traffic.
- Family-selection drift when JSON Validator is chosen from producer class or expectation instead of observed runtime payload type.
- Header/body disagreement risk: trusting `Content-Type` over the observed payload/body can route plain-text or XML payloads into the JSON Validator family incorrectly.
- Scope-violation risk when JSON Validator is attached to DB Tool outputs (JSON Validator workflow is API-client response-output only).
- Missing schema/service-definition configuration when default schema-validation mode is expected.
- User prompt required when schema/service-definition cannot be inferred may pause orchestration until input is provided (intentional blocking behavior).
- Incorrect downgrade risk: using well-formedness mode despite known OpenAPI location and JSON response.
- OpenAPI auto-detect picks the wrong message/schema node for the runtime response when auto-detect is explicitly enabled as an override.
- Incomplete explicit OpenAPI message mapping (for example missing `direction`) causes payload validation setup errors.
- Message mapping is guessed from a paraphrased resource label or concrete runtime URL instead of being reconciled to one confirmed OpenAPI operation template.
- Wrong schema/service-definition inferred from ambiguous context.
- Environment-variable-backed or tool-inferred schema source is applied without explicit user confirmation.
- Synthetic schema generation used instead of user-provided/verified schema location.
- Input-binding mismatch if chained to diagnostics-only output instead of semantic JSON output.
- Approved JSON Validator coverage is silently dropped or replaced with content-only tooling without fresh user approval.
- Experimental-lane drift risk: this card ignores Skill 067's upstream exploration-backed family selection, target parent, or candidate schema-source basis and redoes family-selection work from scratch.

## 9) Safety / Rollback
- Write skill (adds/modifies validator tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<validator-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Technically applicable in Virtualize, but not often used.
- Intended producer class: API client tool response outputs (REST Client now; SOAP/Messaging when corresponding skills are available).
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- When called from Skill 067, this card may rely on upstream exploration-backed payload classification, target parent selection, approved validator-family intent, and candidate schema-source basis, while still owning local validator create/update/readback/verification mechanics.
