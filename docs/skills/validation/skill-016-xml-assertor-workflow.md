# Skill 016: XML Assertor Workflow (Cross-Tool)

## 1) Skill Name
Add, modify, read, delete, copy, and validate XML Assertor on XML-producing tool output

## 2) Objective
Create a reusable lifecycle pattern for XML Assertors on tool output and validate live XPath assertions against runtime traffic/output.

## 3) Scope
- In scope:
  - create XML Assertor via `POST /v6/tools/xmlAssertors`
  - configure assertions via `PUT /v6/tools/xmlAssertors?id=...`
  - readback and verify configuration via `GET /v6/tools/xmlAssertors?id=...`
  - delete XML Assertor via `DELETE /v6/tools?id=...`
  - copy XML Assertor via `POST /v6/tools/copy`
  - run focused execution and validate runtime behavior via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - service/database data remediation
  - UI-only assertion editing workflows

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`

## 4) Inputs
- Required:
  - target output-provider parent id
  - XML Assertor name
  - assertion XPath and expected value
- Optional:
  - `trimContent` option
  - execution filter (`testNames`) for focused runtime checks
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Target test/tool produces XML output at runtime.
- Parent id resolves to an output-provider location that accepts assertor children.
- Runtime response media type for target output is XML (confirmed from baseline run evidence).
- Baseline execution evidence is required before assertion authoring decisions.

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - Construct `parent.id` from Skill 018 mapping; do NOT pass the producer tool id directly.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating the XML Assertor.
  - do not guess or locally invent parent-path mappings.
2. Run baseline execution first and observe live XML payload shape/content from runtime traffic.
  - if observed payload is not XML, stop XML Assertor flow and route validation intent to the appropriate tool family:
    - JSON payload -> Skill 010/029 family
    - plain text payload -> Skill 031 Diff Tool in text mode.
  - if the user explicitly requested XML Assertor but observed payload is not XML, ask for one of two explicit actions before continuing:
    - switch to matching tool family for observed media type, or
    - update producer behavior to emit XML, then rerun baseline.
  - if observed payload is XML, select between XML Assertor and Diff Tool XML mode based on response data volatility:
    - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer XML Assertor with targeted XPath assertions on stable fields,
    - mostly static response data -> prefer Diff Tool XML mode for simpler full-response comparison (see Skill 031),
    - when unsure, start with Diff Tool; switch to XML Assertor if the ignored-differences list grows large.
3. Create XML Assertor:
   - `POST /v6/tools/xmlAssertors`
   - payload must include `parent.id` and typically `name`.
4. Configure a live value assertion:
   - `PUT /v6/tools/xmlAssertors?id=<assertor-id>`
  - for updates to existing XML Assertors, follow read-merge-write (`GET` -> mutate -> `PUT`) per Skill 049.
   - include `toolSettings.assertions[]`, for example:
     - `type: valueAssertion`
     - `selectedElement.xpath`
     - `configuration.expectedValue` (`type: fixed`, `fixed: ...`)
  - when using `hasContentAssertion`, set `selectedElement.extractionType=entireElement` (do not use `contentOnly` for this assertion type).
  - for `regularExpressionAssertion`, treat patterns as full-string match semantics by default:
    - use explicit wildcard wrapping for contains intent (for example `.*token.*`),
    - state case intent explicitly (case-sensitive default unless configured otherwise).
  - for scalar XML value comparisons, apply Skill 054 XPath normalization and use text-node terminal selectors when needed (for example `/account/balance/text()`).
5. Read back assertor to confirm persisted settings.
6. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied assertor via `GET /v6/tools/xmlAssertors?id=<new-id>`.
7. Optional delete flow:
  - `DELETE /v6/tools?id=<assertor-id>` and verify absence in descendants/children.
8. Execute focused verification run and inspect runtime results:
   - if needed, request XML report using `includeXmlReport=true`.
9. If XML parser errors occur (for example `Content is not allowed in prolog`), treat it as input-binding mismatch and re-check the chosen output-provider/input source.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md` (Decision Rule).
- Construct XML Assertor payloads from REST API schema + skill semantics.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Before using it, follow:
- Section 5) Preconditions
- Section 6) Procedure

Rules:
- Replace all `{{...}}` placeholders from runtime evidence + user intent.
- Do not reuse sample ids/names/xpaths/expected values unless explicitly requested.
- For assertion types not shown here, keep the same envelope (`type` + matching assertion object + `selectedElement` + `configuration`) and use OpenAPI (`assertionXml` + matching `<type>Assertion` schema).
- For updates, use `PUT /v6/tools/xmlAssertors?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same assertion envelope shape, no `parent` field).

```json
{
  "parent": {
    "id": "{{PARENT_OUTPUT_PROVIDER_ID}}"
  },
  "name": "{{ASSERTOR_NAME}}",
  "toolSettings": {
    "assertions": [
      {
        "type": "valueAssertion",
        "valueAssertion": {
          "name": "{{ASSERTION_NAME}}",
          "configuration": {
            "expectedValue": {
              "type": "fixed",
              "fixed": "{{EXPECTED_VALUE_FROM_OBSERVED_XML}}"
            }
          },
          "selectedElement": {
            "xpath": "{{XPATH_FROM_OBSERVED_XML}}",
            "extractionType": "contentOnly"
          },
          "options": {
            "trimContent": true
          }
        }
      }
    ],
    "expectedXml": {
      "saveExpectedXml": false
    }
  }
}
```

## 6.1) Composition Guidance (Case-by-Case)
1. Convert the user request into assertion intent:
  - which response element(s) to target
  - what comparison rule to apply (equality/range/pattern/presence)
  - whether expected values are literals or variables from upstream tools
2. Chain XML Assertor under semantic response output anchor:
  - `<producer-tool-id>/Response Traffic` (or other Skill 018-mapped semantic XML output channel)
3. Run a baseline execution first and observe live response payload shape/content from runtime traffic.
  - if observed payload is not XML, stop XML Assertor flow and route validation intent to the appropriate tool family:
    - JSON payload -> Skill 010/029 family
    - plain text payload -> Skill 031 Diff Tool in text mode.
  - if the user explicitly requested XML Assertor but observed payload is not XML, ask for one of two explicit actions before continuing:
    - switch to matching tool family for observed media type, or
    - update producer behavior to emit XML, then rerun baseline.
  - if observed payload is XML, select between XML Assertor and Diff Tool XML mode based on response data volatility:
    - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer XML Assertor with targeted assertions on stable fields,
    - mostly static response data -> prefer Diff Tool XML mode for simpler full-response comparison (see Skill 031),
    - when unsure, start with Diff Tool; switch to XML Assertor if the ignored-differences list grows large.
4. Configure selectors with XPath expressions against observed XML payload structure.
5. Select assertion type to match data semantics:
  - value/parity checks -> `valueAssertion` / `numericAssertion`
  - range checks -> `numericRangeAssertion`
  - string/domain/pattern checks -> `regularExpressionAssertion`
  - presence/shape checks -> `hasContentAssertion` / `occurrenceAssertion`
  - required rule for `hasContentAssertion`:
    - set `selectedElement.extractionType=entireElement` (do not use `contentOnly` for this assertion type)
  - regex assertion rule:
    - treat regex comparisons as full-string match semantics by default,
    - for substring intent, use explicit wildcard patterning (for example `.*token.*`),
    - declare case intent explicitly (case-sensitive default unless configured otherwise).
6. Configure assertions based on user intent against observed payload.
7. Validate with focused verification run and collect run-results-traffic evidence triad.

## 7) Validation
- Creation/update/readback all return `200` on valid payloads.
- Readback contains expected assertion payload:
  - `toolSettings.assertions[].type = valueAssertion`
  - expected XPath and value are present.
- Runtime result confirms assertion behavior on live output (pass/fail with diagnosable message).

## 8) Failure Modes
- `400` for malformed assertion payload.
- `404` for invalid parent/id.
- Runtime XML parse failure indicates non-XML input binding, even when assertion XPath is valid.
- Media-type mismatch risk when XML Assertor is attached to JSON/plain-text traffic.
- False mismatch risk when `hasContentAssertion` uses `selectedElement.extractionType=contentOnly`; use `entireElement` for this assertion type.
- copy placement/name collisions can produce unintended target placement unless readback is performed.
- Over-softening risk: relaxing strict assertion intent after first failure can hide real regressions.

## 8.1) Failure Handling Rule (No Ignore Branch)
- XML Assertor has no ignored-differences feature and should remain strict by default.
- On assertion failure:
  1. summarize mismatch details in human-readable form (XPath, expected vs actual),
  2. confirm with user whether assertion intent or expected value should change,
  3. route deeper runtime diagnosis to:
     - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
- Do not auto-weaken assertions without explicit user confirmation.

## 9) Safety / Rollback
- Write skill (adds/modifies assertor tool nodes).
- Rollback:
  - remove assertor via `DELETE /v6/tools?id=<assertor-id>`
  - or restore previous config using saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Intended producer class: XML-producing semantic output providers (REST Client response output and DB Tool `Results as XML` validated; additional producers when they emit confirmed XML payloads).
- Works best when input-binding path is explicitly verified from runtime artifacts.
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Select semantic output-provider parents (for example response/results) before assertion tuning.
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - Treat Skill 018 as canonical mapping source; update it first when new producer/output types are introduced.
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - Use GET -> mutate -> PUT when editing existing assertors to preserve non-target settings.
