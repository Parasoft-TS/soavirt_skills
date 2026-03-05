# Skill 010: JSON Assertor CRUD + Copy

## 1) Skill Name
JSON Assertor create / modify / delete / copy

## 2) Objective
Manage JSON Assertor tools in `.tst` assets, including create, modify, delete, and copy operations with deterministic parent placement.

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

## 4) Inputs
- Required:
  - target parent id
  - assertor name
- Optional:
  - `toolSettings` assertion definitions
  - `input` values
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Parent tool/suite exists.
- For REST-client chaining, parent must be valid output-provider anchor.
- Runtime response media type for target output is JSON (confirmed from baseline run evidence).
- JSON selector rule must be loaded:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - use XPath-style selectors for JSON fields, not JSONPath.

## 6) Procedure
1. Resolve target context with `GET /v6/children` and/or `GET /v6/descendants/assets`.
2. For REST client chains, derive parent as `<rest-client-id>/Response Traffic` (output provider anchor).
3. Create JSON Assertor with `POST /v6/tools/jsonAssertors` using:
   - `parent.id`
   - `name`
   - optional minimal `toolSettings`
4. Verify with:
   - `GET /v6/descendants/assets?id=<file-id>` for `type=jsonAssertor`
   - `GET /v6/tools/jsonAssertors?id=<created-id>` for full object readback
5. Modify with `PUT /v6/tools/jsonAssertors?id=<id>` (name, input, toolSettings).
6. Delete with `DELETE /v6/tools?id=<id>` and verify absence in descendants.
7. Copy with `POST /v6/tools/copy` (`from.id`, `to.parent.id`, optional `to.name`) and verify new id placement.

## 6.0) Authoring Rule (API-First)
- Construct new JSON Assertor payloads directly from REST API schema + skill semantics.
- Existing JSON Assertor instances in the workspace are optional references for sanity checks only.
- Do not require pre-built examples to author new assertions.

## 6.1) Composition Guidance (Case-by-Case)
1. Convert the user request into assertion intent:
  - which response element(s) to target
  - what comparison rule to apply (equality/range/pattern/presence)
  - whether expected values are literals or variables from upstream tools
2. Chain JSON Assertor under semantic response output anchor:
  - `<rest-client-id>/Response Traffic`
3. Run a baseline execution first and observe live response payload shape/content from runtime traffic.
  - if observed payload is not JSON, stop JSON Assertor flow and route validation intent to the appropriate tool family:
    - XML payload -> Skill 016/030 family
    - plain text payload -> Skill 031 Diff Tool in text mode.
  - if observed payload is JSON, select between JSON Assertor and Diff Tool JSON mode based on response data volatility:
    - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer JSON Assertor with targeted assertions on stable fields,
    - mostly static response data -> prefer Diff Tool JSON mode for simpler full-response comparison (see Skill 031),
    - when unsure, start with Diff Tool; switch to JSON Assertor if the ignored-differences list grows large.
4. Configure selectors with XPath-style expressions (Skill 011), not JSONPath.
5. Select assertion type to match data semantics:
  - numeric parity/range -> `numericAssertion` / `numericRangeAssertion`
  - string/domain/pattern -> `regularExpressionAssertion`
  - presence/shape -> `hasContentAssertion` / `occurrenceAssertion`
6. Configure assertions based on user intent against observed payload.
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
- False mismatch risk from string-vs-number comparisons if assertion type does not match response value type.
- Over-softening risk: auto-relaxing strict assertions after failure can mask real regressions.

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
- Applies to SOAtest: Yes.
- Applies to Virtualize: Not validated.
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
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - Use GET -> mutate -> PUT for updates to existing JSON Assertor tools.

### JSON Tooling Reminder
- For JSON tools in this workspace, selector fields follow XPath-over-JSON semantics.
- Do not use JSONPath in XPath-expected fields.

## 11) Current Validation Status
- **Validated now**:
  - Create JSON Assertor in `/TestAssets/Basic_Test_Construction_Learning.tst` under root REST client by using parent id:
    - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/Simple REST Client - Test In Root Test Suite/Response Traffic`
  - Created tool name:
    - `JSON Assertor - Root Skill Test`
  - Modify flow:
    - Renamed tool to `JSON Assertor - Modified` via `PUT /v6/tools/jsonAssertors?id=...`
  - Assertion configuration flow:
    - Updated `toolSettings.assertions` for `JSON Assertor - Modified` with validated assertion types:
      - `occurrenceAssertion`
      - `numericAssertion`
      - `numericRangeAssertion`
      - `regularExpressionAssertion`
      - `hasContentAssertion`
      - `valueOccurrenceAssertion`
- **Pending**:
  - explicit DELETE flow
  - explicit COPY flow

- **Additional validation example (non-normative)**:
  - parent chain anchor:
    - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/GET Lw==parabankLw==servicesLw==bankLw==accountsLw==${DB_ID}/Response Traffic`
  - created assertor:
    - `JSON Assertor - Balance Matches DB Data Bank`
  - assertion shape:
    - `type=numericAssertion`
    - `selectedElement.xpath=/root/balance` (XPath-style)
    - `expectedValue.fixed=${DB_BALANCE}`
  - focused run evidence:
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_balance_assert_exec_results_1231543615.json`
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_balance_assert_exec_traffic_rest_1231543615.json`
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill1_balance_assert_exec_traffic_db_1231543615.json`
