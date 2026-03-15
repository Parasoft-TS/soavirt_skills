# Skill 010: JSON Assertor CRUD + Copy

## 1) Skill Name
JSON Assertor create / modify / delete / copy

## 2) Objective
Manage JSON Assertor tools in `.tst` assets, including create/modify work plus subordinate copy/delete steps when they are part of broader assertor lifecycle activity.
Pure rename/copy/delete/enable-disable requests on an existing JSON Assertor should route to the centralized operation-centric owners in `docs/skills/structure/skill-068-rename-object.md`, `docs/skills/structure/skill-070-copy-tool.md`, `docs/skills/structure/skill-072-delete-tool.md`, and `docs/skills/structure/skill-074-set-disabled-state.md`.

This skill supports case-by-case assertion authoring composed from user intent (field selection, comparison strategy, and variable usage), rather than fixed domain recipes.

## 3) Scope
- In scope:
  - create via `POST /v6/tools/jsonAssertors`
  - modify via `PUT /v6/tools/jsonAssertors?id=...`
  - delete via `DELETE /v6/tools?id=...`
  - copy via `POST /v6/tools/copy`
  - verify placement via descendants and tool GET
- Out of scope:
  - semantic design of assertion logic content (domain-specific assertion authoring)
  - cross-file migration policies
  - dependency on pre-existing JSON Assertor instances as required creation templates

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`

## 4) Inputs
- Required:
  - target parent id
  - assertor name
- Optional:
  - `dataSource` binding when expected values come from a suite/object datasource column
  - `toolSettings` assertion definitions
  - `input` values
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Parent tool/suite exists.
- For REST-client chaining, parent must be valid output-provider anchor.
- Runtime response media type for target output is JSON and must be confirmed from baseline run evidence before family selection.
- Baseline execution evidence is required before assertion authoring decisions.
- JSON selector rule must be loaded:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - use XPath-style selectors for JSON fields, not JSONPath.

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve target context with deterministic endpoint selection:
  - use `GET /v6/descendants/files?id=<workspace-folder>&type=tst` to resolve target `.tst` file id when needed,
  - use `GET /v6/children?id=<tst-id>` for root suite context,
  - use `GET /v6/descendants/assets?id=<tst-id>` or `GET /v6/descendants/assets?id=<suite-id>` to resolve nested suite/tool targets.
  - do not use `GET /v6/descendants/assets` with directory ids.
2. Resolve the target producer/output-provider parent from `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5), then construct `parent.id` from that mapping.
2.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating/modifying JSON Assertor chains.
  - do not guess or locally invent parent-path mappings.
2.2 Fail-closed media-type gate:
  - run baseline execution and inspect the observed semantic response payload/body first; use response headers such as `Content-Type` only as supporting evidence.
  - if observed payload is JSON, continue JSON Assertor flow.
  - if observed payload is XML, route to Skill 016/030 instead of creating/updating JSON Assertor.
  - if observed payload is plain text, route to Skill 031 in text mode instead of creating/updating JSON Assertor.
  - do not select JSON Assertor only because the producer is a REST Client or other API client tool.
  - if the caller/orchestration already approved JSON Assertor for this target, treat that family as binding and do not reopen JSON Assertor vs Diff Tool selection inside this card.
3. Create JSON Assertor with `POST /v6/tools/jsonAssertors` using:
   - `parent.id`
   - `name`
   - optional minimal `toolSettings`
4. Verify with:
   - `GET /v6/descendants/assets?id=<file-id>` for `type=jsonAssertor`
   - `GET /v6/tools/jsonAssertors?id=<created-id>` for full object readback
5. Modify with `PUT /v6/tools/jsonAssertors?id=<id>` (name, input, toolSettings).
  - for updates to existing JSON Assertors, follow read-merge-write (`GET` -> mutate -> `PUT`) per Skill 049.
6. Delete flow (substep only):
  - for pure delete-only intent, prefer `docs/skills/structure/skill-072-delete-tool.md`
  - keep this branch only when delete is subordinate to broader JSON Assertor lifecycle/configuration work
  - `DELETE /v6/tools?id=<id>` and verify absence in descendants.
7. Copy flow (substep only):
  - for pure copy-only intent, prefer `docs/skills/structure/skill-070-copy-tool.md`
  - keep this branch only when copy is subordinate to broader JSON Assertor lifecycle/configuration work
  - `POST /v6/tools/copy` (`from.id`, `to.parent.id`, optional `to.name`) and verify new id placement.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md` (Decision Rule).
- Construct JSON Assertor payloads from REST API schema + skill semantics.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Before using it, follow:
- Section 5) Preconditions
- Section 6) Procedure

Rules:
- Replace all `{{...}}` placeholders from runtime evidence + user intent.
- Do not reuse sample ids/names/xpaths/expected values unless explicitly requested.
- For assertion types not shown here, keep the same envelope (`type` + matching assertion object + `selectedElement` + `configuration`) and use OpenAPI (`assertionJson` + matching `<type>Assertion` schema).
- For less-common assertion families, confirm both the enum name and the matching assertion/configuration schema in OpenAPI before authoring. The cached spec currently models some families more completely than others.
- For updates, use `PUT /v6/tools/jsonAssertors?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same assertion envelope shape, no `parent` field).
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
        "type": "hasContentAssertion",
        "hasContentAssertion": {
          "name": "{{ASSERTION_NAME}}",
          "configuration": {
            "hasContent": {
              "type": "fixed",
              "fixed": "true"
            }
          },
          "selectedElement": {
            "xpath": "{{XPATH_FROM_OBSERVED_JSON}}",
            "extractionType": "entireElement"
          },
          "options": {
            "trimContent": true
          }
        }
      }
    ],
    "expectedJson": {
      "saveExpectedJson": false
    }
  }
}
```

## 6.1) Composition Guidance (Case-by-Case)
1. Convert the user request into assertion intent:
  - which response element(s) to target
  - what comparison rule to apply (equality/range/pattern/presence)
  - whether expected values are literals or variables from upstream tools
2. Chain JSON Assertor under semantic response output anchor:
  - `<rest-client-id>/Response Traffic`
3. Apply the fail-closed media-type gate from Procedure step 2.2 before authoring assertion logic.
  - if observed payload is JSON and family selection has not already been fixed by the caller, select between JSON Assertor and Diff Tool JSON mode based on response data volatility:
    - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer JSON Assertor with targeted assertions on stable fields,
    - mostly static response data -> prefer Diff Tool JSON mode for simpler full-response comparison (see Skill 031),
    - when unsure, start with Diff Tool; switch to JSON Assertor if the ignored-differences list grows large.
  - if the caller/orchestration already approved JSON Assertor for this target, execute that approved assertor branch rather than substituting Diff Tool locally.
4. Configure selectors with XPath-style expressions (Skill 011), not JSONPath.
  - Skill 054 boundary: XML scalar text-node normalization (`/text()`) is not required for JSON selectors; keep XPath-over-JSON selector form.
5. Select assertion type to match data semantics:
  - string/domain/pattern -> `stringComparisonAssertion` / `regularExpressionAssertion`
  - numeric parity/range -> `numericAssertion` / `numericRangeAssertion`
  - presence/shape -> `hasContentAssertion` / `occurrenceAssertion` / `hasChildrenAssertion` / `typeAssertion`
  - date/date-time range windows -> `dateRangeAssertion` / `dateTimeRangeAssertion`
  - required rule for `hasContentAssertion`:
    - set `selectedElement.extractionType=entireElement` (do not use `contentOnly` for this assertion type)
  - regex assertion rule:
    - treat regex comparisons as full-string match semantics by default,
    - for substring intent, use explicit wildcard patterning (for example `.*12212.*`),
    - declare case intent explicitly (case-sensitive default unless configured otherwise).
  - current cached OpenAPI note:
    - `numericDifferenceAssertion`, `dateDifferenceAssertion`, and `dateTimeDifferenceAssertion` appear in the JSON assertion-type enum, but the cached schema does not expose matching assertion/configuration objects in `assertionJson`; treat those families as contract-ambiguous until the server schema is clarified.
6. Configure assertions based on user intent against observed payload.
  - when expected values come from datasource columns, set tool-level `dataSource` to the datasource name available in the current object context and encode expected-value fields as parameterized column references rather than fixed `${column}` literals.
7. Validate with focused verification run and collect run-results-traffic evidence triad.

## 7) Validation
- Expected HTTP status codes:
  - `200` on success
  - `400` for invalid parent/payload
- Post-condition checks:
  - created/copied assertor appears under expected parent path
  - updated fields persist after GET
  - deleted assertor no longer present in descendants

## 8) Failure Modes
- `400` on create when parent id is incorrect (for REST clients, parent must be output-provider path, not plain tool id).
- Silent misplacement risk if parent anchor is guessed incorrectly.
- Copy/move conflicts on duplicate names in same parent.
- Selector mismatch risk if JSONPath syntax is used in selector fields that require XPath-style expressions.
- Media-type mismatch risk when JSON Assertor is attached to XML/plain-text traffic.
- Header/body disagreement risk: trusting `Content-Type` over the observed payload/body can route plain-text or XML payloads into the JSON Assertor family incorrectly.
- False mismatch risk from string-vs-number comparisons if assertion type does not match response value type.
- False mismatch risk when `hasContentAssertion` uses `selectedElement.extractionType=contentOnly`; use `entireElement` for this assertion type.
- `Variable "<column>" could not be resolved` usually means a datasource-backed expectation was modeled as a fixed literal/variable string instead of a parameterized datasource column reference.
- `No Data Source column named <column>` usually means tool-level `dataSource` is missing, uses the wrong datasource name for the current object context, or uses a full asset id where the tool expects the local datasource name.
- Secondary numeric/string formatting errors can appear after datasource-binding failures and should not be treated as the primary root cause until binding is corrected.
- Over-softening risk: auto-relaxing strict assertions after failure can mask real regressions.
- Approved JSON Assertor intent can be silently weakened if this card reopens Assertor-vs-Diff selection after orchestration approval.

## 8.1) Failure Handling Rule (No Ignore Branch)
- JSON Assertor has no ignored-differences feature and should remain strict by default.
- On assertion failure:
  1. summarize human-readable mismatch details (field, expected vs actual, assertion type),
  2. ask user whether expected behavior or assertion intent should change,
  3. if deeper triage is needed, hand off to diagnostics workflow:
     - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
- Do not auto-weaken assertions without explicit user confirmation.

## 9) Safety / Rollback
- Read-only by default? No (write skill).
- Rollback plan:
  - delete created/copied tools for cleanup
  - restore file snapshot if broad unintended edits occur

## 10) Reuse Notes
- Primary target: SOAtest.
- Technically applicable in Virtualize, but not often used.
- Intended producer class: JSON-producing semantic output providers (REST Client response output now; other producer outputs only when they emit confirmed JSON payloads).
- Shared components:
  - `POST/PUT/GET /v6/tools/jsonAssertors`
  - `DELETE /v6/tools`
  - `POST /v6/tools/copy`
  - `GET /v6/children`, `GET /v6/descendants/assets`
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - Use XPath-style selectors (for example `/root/item/type`) in selector fields even for JSON payload validation.
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Prefer semantic response/results output-provider anchors over diagnostics-first traffic outputs.
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- For pure rename/copy/delete/enable-disable prompts on an existing JSON Assertor, prefer the centralized operation-centric owners; keep Skill 010 for broader assertor lifecycle/configuration work.
