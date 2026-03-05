# Skill 030: XML Validator Workflow (Cross-Tool)

## 1) Skill Name
Add and validate XML Validator on XML-producing tool output

## 2) Objective
Create a reusable workflow for chaining an XML Validator to XML output, defaulting to schema validation with an explicit schema location/service-definition-derived schema configuration, and validating runtime payload structure/content against validator configuration.

Important: schema-validation mode is only considered correctly configured when the referenced-schema option is enabled so the WSDL/Schema location field is actively used.

## 3) Scope
- In scope:
  - create XML Validator via `POST /v6/tools/xmlValidators`
  - configure XML Validator via `PUT /v6/tools/xmlValidators?id=...`
  - readback via `GET /v6/tools/xmlValidators?id=...`
  - run focused execution and validate runtime behavior via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - auto-remediation of payload/schema issues
  - domain-specific XML schema design guidance

## 4) Inputs
- Required:
  - target output-provider parent id that emits XML
  - XML Validator name
  - schema configuration for validation (for example schema location)
- Optional:
  - explicit validator configuration settings (mode, schema/service-definition location)
  - schema/service-definition hints from prior user prompts
  - schema/service-definition hints from environment variables
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits XML payloads.
- Parent id resolves to an output-provider location that accepts chained tools.
- Runtime response media type for target output is XML (confirmed from baseline run evidence).

## 6) Procedure
1. Construct the output-provider parent path for the target producer tool:
  - REST Client: `<rest-client-id>/Response Traffic`
  - DB Tool: `<db-tool-id>/Results as XML`
  - Do NOT pass the producer tool id directly as `parent.id`; tool creation requires `parent.id` of type `outputProvider` or `suite`. Passing a REST Client id (type `REST Client`) returns `400`.
  - Do NOT attempt to discover output provider ids via `/children` or `/descendants/assets`; empty output providers are not returned. Construct the path directly.
  - See `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` Section 5 for canonical patterns.
1.1 Run baseline execution and confirm observed response media type is XML before creating/updating XML Validator.
  - if payload is JSON, route to Skill 029.
  - if payload is plain text, route to Skill 031 (text mode).
2. Resolve validator identity before writes (idempotent upsert rule):
  - derive deterministic target id as `<parent-id>/<validator-name>`,
  - if that id exists, update it in place,
  - create via `POST /v6/tools/xmlValidators` only when validator is truly absent.
  - do not create another validator with the same intent/name in the same parent.
3. Build candidate schema/service-definition inputs using this precedence:
  - explicit values provided in the current user request,
  - schema/service-definition values stated earlier in the current session,
  - relevant environment variables already configured for this test context.
4. Apply confidence gate before configuring schema validation:
  - proceed only when one candidate is unambiguous and format-compatible with the XML payload,
  - if candidates conflict, are missing, or are weakly inferred, ask a targeted user question and wait for confirmation.
  - do not generate synthetic/placeholder schemas to bypass missing schema-location input.
5. Create XML Validator when absent:
   - `POST /v6/tools/xmlValidators` with `parent.id` and `name`.
6. Configure validator settings:
   - `PUT /v6/tools/xmlValidators?id=<validator-id>`.
  - default to `toolSettings.mode=validateAgainstSchema`.
  - set `toolSettings.schemaValidationOptions.validateAgainstSchemasReferenced=true`.
  - set schema configuration (for example `toolSettings.schemaValidationOptions.location`) from explicit or high-confidence inferred inputs.
  - when schema/service-definition binding references operation signatures, keep template paths/placeholders from the definition model (for example `{customerId}`) rather than concretized runtime values.
  - use `checkWellFormednessOnly` only when user explicitly requests it.
7. Read back tool and verify configuration persisted.
8. Execute focused run and inspect runtime results.

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback includes persisted validator settings.
- Repeat executions/update passes do not create additional validators for the same parent/name intent.
- Readback confirms default mode is `validateAgainstSchema` unless user explicitly selected `checkWellFormednessOnly`.
- Readback confirms `toolSettings.schemaValidationOptions.validateAgainstSchemasReferenced=true` when schema-validation mode is active.
- Inferred schema/service-definition values are only applied when confidence gate conditions are met.
- Runtime result shows validator pass/fail aligned to payload validity and configuration.

## 8) Failure Modes
- `400` invalid payload shape or unsupported validator settings.
- `404` invalid tool id or parent id.
- XML parser/prolog errors from non-XML input binding.
- Media-type mismatch when XML Validator is attached to JSON/plain-text traffic.
- Missing schema configuration when default schema-validation mode is expected.
- Referenced-schema option disabled, causing WSDL/Schema location to be ignored.
- Wrong schema/service-definition inferred from ambiguous context.
- Synthetic schema generation used instead of user-provided/verified schema location.
- Input-binding mismatch if chained to diagnostics-only output instead of semantic XML output.

## 9) Safety / Rollback
- Write skill (adds/modifies validator tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<validator-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Applies to SOAtest: expected; runtime validation pending first dedicated run.
- Applies to Virtualize: not yet validated.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`

## 11) Initial Validation Plan
1. Create under XML-producing output provider.
2. Resolve schema/service-definition using precedence + confidence gate (ask user when uncertain).
3. Configure baseline validation settings with default `validateAgainstSchema` mode.
4. Execute focused run and capture pass/fail behavior.
