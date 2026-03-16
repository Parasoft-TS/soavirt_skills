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
  - run focused execution and validate runtime behavior by following the Skill 012 execution-triad contract
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
- When called from Skill 067, this card may rely on upstream exploration-backed payload classification, target parent selection, approved validator-family intent, and candidate schema-source basis, while still owning local validator create/update/readback/verification mechanics.

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
- In the stable lane, runtime response media type for target output is XML and must be confirmed from baseline run evidence before family selection. When this card is called from Skill 067 in the experimental lane, upstream exploration-backed payload classification, approved validator family, target parent selection, and candidate schema-source basis may satisfy that family-selection gate before attachment, but post-attachment focused verification remains required.

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
1.2 Fail-closed media-type gate:
  - in the stable lane, run baseline execution and inspect the observed semantic response payload/body first; use response headers such as `Content-Type` only as supporting evidence.
  - when this card is called from Skill 067, accept upstream exploration-backed payload classification, approved validator-family intent, target parent selection, and candidate schema-source basis as authoritative for family-selection purposes instead of rerunning baseline work only to rediscover them.
  - if observed or upstream-authoritative payload is XML, continue XML Validator flow.
  - if observed or upstream-authoritative payload is JSON, route to Skill 029 instead of creating/updating XML Validator.
  - if observed or upstream-authoritative payload is plain text, route to Skill 031 in text mode instead of creating/updating XML Validator.
  - do not select XML Validator only because the producer is a REST Client or other API client tool.
2. Resolve validator identity before writes (idempotent upsert rule):
  - derive deterministic target id as `<parent-id>/<validator-name>`,
  - if that id exists, update it in place,
  - create via `POST /v6/tools/xmlValidators` only when validator is truly absent.
  - do not create another validator with the same intent/name in the same parent.
2.1 Approved-plan preservation rule:
  - if the caller/orchestration already approved XML Validator for this target, treat validator coverage as a required deliverable for that target.
  - if XML Validator creation or configuration cannot proceed, mark the target `blocked` or `partial` and return for a new user decision rather than silently continuing with only Diff/Assertor coverage.
3. Build candidate schema/service-definition inputs using this precedence:
  - explicit values provided in the current user request,
  - schema/service-definition values stated earlier in the current session,
  - relevant environment variables already configured for this test context,
  - inferable schema/service-definition hints from existing tools and their children in the target `.tst` (for example generated suite/source naming and validator-capable sibling configuration).
3.1. Preserve candidate provenance while collecting inputs:
  - if a candidate comes from an environment variable, retain both the variable form and the resolved value,
  - if a candidate is inferred from tool/tree context, record the exact source that made it plausible.
4. Apply confidence + confirmation gate before configuring schema validation:
  - proceed only when one candidate is unambiguous, format-compatible with the XML payload, and explicitly confirmed by the user unless the user already provided that exact source in the current request,
  - if candidates conflict, are missing, weakly inferred, or only environment/tool-context derived, ask a targeted user question and wait for confirmation before use,
  - when confirming an environment-variable-backed source, echo both the variable form and the resolved value.
  - do not generate synthetic/placeholder schemas to bypass missing schema-location input.
  - do not create/configure XML Validator in `checkWellFormednessOnly` mode because definition inputs are unknown.
4.1 Missing-definition prompt rule (required):
  - when schema/service-definition inputs cannot be inferred from user input, session context, environment variables, or tool/tree inspection with high confidence, prompt the user to provide schema/service-definition location.
  - treat this as a blocking gate for schema-mode configuration; wait for user input before completing validator configuration.
  - even high-confidence environment-variable-backed or tool/tree-inferred candidates still require explicit user confirmation before use.
  - do not default to `checkWellFormednessOnly` only because definition inputs are missing.
  - only default to `checkWellFormednessOnly` if the user explicitly agrees to that mode.
5. Create XML Validator when absent:
   - `POST /v6/tools/xmlValidators` with `parent.id` and `name`.
6. Configure validator settings:
   - `PUT /v6/tools/xmlValidators?id=<validator-id>`.
  - default to `toolSettings.mode=validateAgainstSchema`.
  - set `toolSettings.schemaValidationOptions.validateAgainstSchemasReferenced=true`.
  - set schema configuration (for example `toolSettings.schemaValidationOptions.location`) from explicit or explicitly confirmed inferred inputs.
  - when schema/service-definition binding references operation signatures, keep template paths/placeholders from the definition model (for example `{customerId}`) rather than concretized runtime values.
  - use `checkWellFormednessOnly` only when user explicitly requests it.
7. Read back tool and verify configuration persisted.
8. Execute focused run by following the Skill 012 execution-triad contract and inspect runtime results.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md` (Decision Rule).
- Construct XML Validator payloads from REST API schema + skill semantics.

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
- Inferred schema/service-definition values are only applied when confidence gate conditions are met and the user explicitly confirms the candidate source.
- When definition inputs are not inferable with high confidence, workflow blocks on user prompt rather than downgrading validation mode.
- Environment-variable-backed candidates are echoed using both variable form and resolved value before they are used.
- Runtime result shows validator pass/fail aligned to payload validity and configuration.
- When XML Validator coverage was approved by the caller/orchestration, it is either delivered or explicitly reported as blocked/partial; it is not silently omitted.

## 8) Failure Modes
- `400` invalid payload shape or unsupported validator settings.
- `404` invalid tool id or parent id.
- XML parser/prolog errors from non-XML input binding.
- Media-type mismatch when XML Validator is attached to JSON/plain-text traffic.
- Family-selection drift when XML Validator is chosen from producer class or expectation instead of observed runtime payload type.
- Header/body disagreement risk: trusting `Content-Type` over the observed payload/body can route JSON/plain-text payloads into the XML Validator family incorrectly.
- Scope-violation risk when XML Validator is attached to DB Tool outputs (XML Validator workflow is API-client response-output only).
- Missing schema configuration when default schema-validation mode is expected.
- User prompt required when schema/service-definition cannot be inferred may pause orchestration until input is provided (intentional blocking behavior).
- Referenced-schema option disabled, causing WSDL/Schema location to be ignored.
- Incorrect downgrade risk: using well-formedness mode despite available/required schema validation intent.
- Wrong schema/service-definition inferred from ambiguous context.
- Environment-variable-backed or tool-inferred schema source is applied without explicit user confirmation.
- Synthetic schema generation used instead of user-provided/verified schema location.
- Input-binding mismatch if chained to diagnostics-only output instead of semantic XML output.
- Approved XML Validator coverage is silently dropped or replaced with content-only tooling without fresh user approval.
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
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- When called from Skill 067, this card may rely on upstream exploration-backed payload classification, target parent selection, approved validator-family intent, and candidate schema-source basis, while still owning local validator create/update/readback/verification mechanics.
