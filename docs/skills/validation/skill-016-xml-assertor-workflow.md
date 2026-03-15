# Skill 016: XML Assertor Workflow (Cross-Tool)

## 1) Skill Name
Add, modify, read, delete, copy, and validate XML Assertor on XML-producing tool output

## 2) Objective
Create a reusable lifecycle pattern for XML Assertors on tool output and validate live XPath assertions against runtime traffic/output, while leaving pure rename/copy/delete/enable-disable requests to the centralized operation-centric owners.

## 3) Scope
- In scope:
  - create XML Assertor via `POST /v6/tools/xmlAssertors`
  - configure assertions via `PUT /v6/tools/xmlAssertors?id=...`
  - readback and verify configuration via `GET /v6/tools/xmlAssertors?id=...`
  - delete XML Assertor via `DELETE /v6/tools?id=...`
  - copy XML Assertor via `POST /v6/tools/copy`
  - run focused execution and validate runtime behavior by following the Skill 012 execution-triad contract
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
  - `dataSource` binding when expected values come from a suite/object datasource column
  - `trimContent` option
  - execution filter (`testNames`) for focused runtime checks
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Target test/tool produces XML output at runtime.
- Parent id resolves to an output-provider location that accepts assertor children.
- Runtime response media type for target output is XML and must be confirmed from baseline run evidence before family selection.
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
1.2 Fail-closed media-type gate:
  - run baseline execution and inspect the observed semantic response payload/body first; use response headers such as `Content-Type` only as supporting evidence.
  - if observed payload is XML, continue XML Assertor flow.
  - if observed payload is JSON, route to Skill 010/029 instead of creating/updating XML Assertor.
  - if observed payload is plain text, route to Skill 031 in text mode instead of creating/updating XML Assertor.
  - do not select XML Assertor only because the producer is a REST Client, DB Tool, or other tool that often emits XML.
  - if the caller/orchestration already approved XML Assertor for this target, treat that family as binding and do not reopen XML Assertor vs Diff Tool selection inside this card.
2. If observed payload is XML and family selection has not already been fixed by the caller, select between XML Assertor and Diff Tool XML mode based on response data volatility:
  - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer XML Assertor with targeted XPath assertions on stable fields,
  - mostly static response data -> prefer Diff Tool XML mode for simpler full-response comparison (see Skill 031),
  - when unsure, start with Diff Tool; switch to XML Assertor if the ignored-differences list grows large.
  - if the caller/orchestration already approved XML Assertor for this target, execute that approved assertor branch rather than substituting Diff Tool locally.
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
  - when expected values come from datasource columns:
    - set tool-level `dataSource` to the datasource name accepted in the current object context,
    - encode `configuration.expectedValue` as `type=parameterized` with `parameterized.columnName=<column>`,
    - do not encode datasource columns as fixed `${column}` literals,
    - do not assume the full datasource asset id is accepted when the tool only exposes local datasource names.
  - when using `hasContentAssertion`, set `selectedElement.extractionType=entireElement` (do not use `contentOnly` for this assertion type).
  - for `regularExpressionAssertion`, treat patterns as full-string match semantics by default:
    - use explicit wildcard wrapping for contains intent (for example `.*token.*`),
    - state case intent explicitly (case-sensitive default unless configured otherwise).
  - for scalar XML value comparisons, apply Skill 054 XPath normalization and use text-node terminal selectors when needed (for example `/account/balance/text()`).
5. Read back assertor to confirm persisted settings.
6. Optional copy flow (substep only):
  - for pure copy-only intent, prefer `docs/skills/structure/skill-070-copy-tool.md`
  - keep this branch only when copy is subordinate to broader XML Assertor lifecycle/configuration work
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied assertor via `GET /v6/tools/xmlAssertors?id=<new-id>`.
7. Optional delete flow (substep only):
  - for pure delete-only intent, prefer `docs/skills/structure/skill-072-delete-tool.md`
  - keep this branch only when delete is subordinate to broader XML Assertor lifecycle/configuration work
  - `DELETE /v6/tools?id=<assertor-id>` and verify absence in descendants/children.
8. Execute focused verification run by following the Skill 012 execution-triad contract and inspect runtime results.
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
- For less-common assertion families, confirm both the enum name and the matching assertion/configuration schema in OpenAPI before authoring. The cached spec currently models some families more completely than others.
- For updates, use `PUT /v6/tools/xmlAssertors?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same assertion envelope shape, no `parent` field).
- For datasource-backed expected values, set tool-level `dataSource` to the datasource name accepted in the current object context and encode expected-value fields as `type=parameterized` with `parameterized.columnName=<column>`; do not encode datasource columns as fixed `${column}` literals.

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
3. Apply the fail-closed media-type gate from Procedure step 1.2 before authoring assertion logic.
4. Configure selectors with XPath expressions against observed XML payload structure.
5. Select assertion type to match data semantics:
  - string/domain/pattern -> `stringComparisonAssertion` / `regularExpressionAssertion`
  - numeric parity/range -> `numericAssertion` / `numericRangeAssertion`
  - presence/shape -> `hasContentAssertion` / `occurrenceAssertion` / `hasChildrenAssertion`
  - date/date-time range windows -> `dateRangeAssertion` / `dateTimeRangeAssertion`
  - required rule for `hasContentAssertion`:
    - set `selectedElement.extractionType=entireElement` (do not use `contentOnly` for this assertion type)
  - regex assertion rule:
    - treat regex comparisons as full-string match semantics by default,
    - for substring intent, use explicit wildcard patterning (for example `.*token.*`),
    - declare case intent explicitly (case-sensitive default unless configured otherwise).
  - current cached OpenAPI note:
    - `typeAssertion` is modeled for JSON, but is not exposed in the XML assertion enum/object schema.
    - `numericDifferenceAssertion`, `dateDifferenceAssertion`, and `dateTimeDifferenceAssertion` appear in the XML assertion-type enum, but the cached schema does not expose matching assertion/configuration objects in `assertionXml`; treat those families as contract-ambiguous until the server schema is clarified.
6. Configure assertions based on user intent against observed payload.
  - when expected values come from datasource columns, set tool-level `dataSource` to the datasource name available in the current object context and encode expected-value fields as parameterized column references rather than fixed `${column}` literals.
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
- Header/body disagreement risk: trusting `Content-Type` over the observed payload/body can route JSON/plain-text payloads into the XML Assertor family incorrectly.
- False mismatch risk when `hasContentAssertion` uses `selectedElement.extractionType=contentOnly`; use `entireElement` for this assertion type.
- `Variable "<column>" could not be resolved` usually means a datasource-backed expectation was modeled as a fixed literal/variable string instead of a parameterized datasource column reference.
- `No Data Source column named <column>` usually means tool-level `dataSource` is missing, uses the wrong datasource name for the current object context, or uses a full asset id where the tool expects the local datasource name.
- Secondary numeric/string formatting errors can appear after datasource-binding failures and should not be treated as the primary root cause until binding is corrected.
- copy placement/name collisions can produce unintended target placement unless readback is performed.
- Over-softening risk: relaxing strict assertion intent after first failure can hide real regressions.
- Approved XML Assertor intent can be silently weakened if this card reopens Assertor-vs-Diff selection after orchestration approval.

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
- Technically applicable in Virtualize, but not often used.
- Intended producer class: XML-producing semantic output providers (REST Client response output and DB Tool `Results as XML` validated; additional producers when they emit confirmed XML payloads).
- Works best when input-binding path is explicitly verified from runtime artifacts.
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Select semantic output-provider parents (for example response/results) before assertion tuning.
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- For pure rename/copy/delete/enable-disable prompts on an existing XML Assertor, prefer the centralized operation-centric owners; keep Skill 016 for broader assertor lifecycle/configuration work.
