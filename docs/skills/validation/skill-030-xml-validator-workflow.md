# Skill 030: XML Validator Workflow (Cross-Tool)

## 1) Skill Name
Add and validate XML Validator on XML-producing tool output

## 2) Objective
Create a reusable workflow for chaining an XML Validator to API client XML response output, defaulting to schema validation with an explicit schema location/service-definition-derived schema configuration, and validating runtime payload structure/content against validator configuration.

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

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`

## 4) Inputs
- Required:
  - target output-provider parent id that emits XML
  - XML Validator name
  - schema/service-definition location (for example XSD/WSDL URL or resolvable file/id) for schema validation configuration
- Optional:
  - explicit validator configuration settings (mode, schema/service-definition location)
  - schema/service-definition hints from prior user prompts
  - schema/service-definition hints from environment variables
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits XML payloads.
- Parent id resolves to an output-provider location that accepts chained tools.
- Target producer is an API client tool response output (REST Client today; SOAP/Messaging client outputs when those skills are available).
- Runtime response media type for target output is XML (confirmed from baseline run evidence).

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - This workflow remains API-client response-output scoped; do not target DB Tool outputs for XML Validator schema-validation workflows.
  - Construct `parent.id` from Skill 018 mapping.
  - Do NOT pass the producer tool id directly as `parent.id`; tool creation requires `parent.id` of type `outputProvider` or `suite`. Passing a REST Client id (type `REST Client`) returns `400`.
  - Do NOT attempt to discover output provider ids via `/children` or `/descendants/assets`; empty output providers are not returned.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating/configuring XML Validator.
  - do not guess or locally invent parent-path mappings.
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
  - relevant environment variables already configured for this test context,
  - inferable schema/service-definition hints from existing tools and their children in the target `.tst` (for example generated suite/source naming and validator-capable sibling configuration).
4. Apply confidence gate before configuring schema validation:
  - proceed only when one candidate is unambiguous and format-compatible with the XML payload,
  - if candidates conflict, are missing, or are weakly inferred, ask a targeted user question and wait for confirmation.
  - do not generate synthetic/placeholder schemas to bypass missing schema-location input.
  - do not create/configure XML Validator in `checkWellFormednessOnly` mode because definition inputs are unknown.
4.1 Missing-definition prompt rule (required):
  - when schema/service-definition inputs cannot be inferred from user input, session context, environment variables, or tool/tree inspection with high confidence, prompt the user to provide schema/service-definition location.
  - treat this as a blocking gate for schema-mode configuration; wait for user input before completing validator configuration.
  - do not default to `checkWellFormednessOnly` only because definition inputs are missing.
  - only default to `checkWellFormednessOnly` if the user explicitly agrees to that mode.
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

## 6.0) Authoring Rule (API-First)
- Construct new XML Validator payloads directly from REST API schema + skill semantics.
- Existing XML Validator instances in the workspace are optional references for sanity checks only.
- Do not require pre-built examples to author validator configuration.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Before using it, follow:
- Section 5) Preconditions
- Section 6) Procedure

Rules:
- Replace all `{{...}}` placeholders using runtime evidence + user intent.
- Do not reuse sample ids/names/locations unless explicitly requested.
- Keep schema-validation options explicit when schema mode is used (including `validateAgainstSchemasReferenced`).
- For updates, use `PUT /v6/tools/xmlValidators?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same `toolSettings` shape, no `parent` field).

```json
{
  "parent": {
    "id": "{{PARENT_OUTPUT_PROVIDER_ID}}"
  },
  "name": "{{XML_VALIDATOR_NAME}}",
  "toolSettings": {
    "mode": "validateAgainstSchema",
    "resolveExternalDtds": true,
    "schemaValidationOptions": {
      "useNamespaceAsLocationURI": true,
      "validateAgainstSchemasReferenced": true,
      "location": {
        "external": "{{XSD_OR_WSDL_URL}}"
      }
    }
  }
}
```

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback includes persisted validator settings.
- Repeat executions/update passes do not create additional validators for the same parent/name intent.
- Readback confirms default mode is `validateAgainstSchema` unless user explicitly selected `checkWellFormednessOnly`.
- Readback confirms `toolSettings.schemaValidationOptions.validateAgainstSchemasReferenced=true` when schema-validation mode is active.
- Inferred schema/service-definition values are only applied when confidence gate conditions are met.
- When definition inputs are not inferable with high confidence, workflow blocks on user prompt rather than downgrading validation mode.
- Runtime result shows validator pass/fail aligned to payload validity and configuration.

## 8) Failure Modes
- `400` invalid payload shape or unsupported validator settings.
- `404` invalid tool id or parent id.
- XML parser/prolog errors from non-XML input binding.
- Media-type mismatch when XML Validator is attached to JSON/plain-text traffic.
- Scope-violation risk when XML Validator is attached to DB Tool outputs (XML Validator workflow is API-client response-output only).
- Missing schema configuration when default schema-validation mode is expected.
- User prompt required when schema/service-definition cannot be inferred may pause orchestration until input is provided (intentional blocking behavior).
- Referenced-schema option disabled, causing WSDL/Schema location to be ignored.
- Incorrect downgrade risk: using well-formedness mode despite available/required schema validation intent.
- Wrong schema/service-definition inferred from ambiguous context.
- Synthetic schema generation used instead of user-provided/verified schema location.
- Input-binding mismatch if chained to diagnostics-only output instead of semantic XML output.

## 9) Safety / Rollback
- Write skill (adds/modifies validator tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<validator-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Intended producer class: API client tool response outputs (REST Client now; SOAP/Messaging when corresponding skills are available).
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - when new producer/output types are introduced, update Skill 018 first, then apply here.
