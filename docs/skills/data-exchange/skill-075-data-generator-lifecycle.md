# Data Generator Tool Lifecycle Skill Card
## Update Policy
Updates to Data Generator Tool objects must preserve non-targeted extracts, destination wiring, and opaque top-level fields through GET -> mutate -> PUT read-merge-write. See Skill 017 for downstream output chaining, Skill 049 for tool PUT mutation safety, and Skill 050 for capability preflight.
## Capability Preflight
Before performing any mutation or lifecycle operation, perform capability preflight checks as described in Skill 050.
## Cross-Cutting Dependencies
Depends on:
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md` (downstream output chaining)
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` (read-merge-write update)
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` (capability preflight)
- `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md` (destination-column discovery when writable targets are not already confirmed)
# Skill 075: Data Generator Tool Lifecycle (Create/Update/Copy/Move/Delete)
## 1) Skill Name
Data Generator Tool create / update / copy / move / delete
## 2) Objective
Manage SOAtest Data Generator Tools in `.tst` assets with deterministic placement and explicit generator-to-destination mapping for suite variables, custom columns, or writable datasource columns. Pure rename-only requests on existing Data Generator Tools should route to `docs/skills/structure/skill-068-rename-object.md`, and pure copy/move/delete requests may route to the generic tool lifecycle leaves when no broader Data Generator configuration change is in scope.
## 3) Scope
- In scope:
  - create Data Generator Tool via `POST /v6/tools/dataGenerators`
  - update Data Generator Tool via `PUT /v6/tools/dataGenerators?id=...`
  - read Data Generator Tool via `GET /v6/tools/dataGenerators?id=...`
  - copy/move Data Generator Tool via `POST /v6/tools/copy` and `POST /v6/tools/move`
  - delete Data Generator Tool via `DELETE /v6/tools?id=...`
  - positional placement under suites using `PUT /v6/suites/children?id=...`
- Out of scope:
  - downstream consumer authoring beyond output-chaining guidance
  - inventing destination columns, suite variable names, or generator defaults from prior examples
  - local-YAML-first edits for ordinary lifecycle work
## 4) Inputs
- Required:
  - target parent suite id(s)
  - at least one extract under `toolSettings.dataGenerators[]`
- Required per extract:
  - extract `name`
  - exactly one destination branch under `dataSourceColumn`:
    - `customColumn.customColumnName`
    - `suiteVariable.variable`
    - `writableDataSource.writableDataSourceColumn`
  - generator `type`: `string`, `number`, or `date`
  - generator-type settings sufficient to express the requested behavior
- Optional:
  - Data Generator Tool name
  - top-level `dataSource` when user explicitly wants it set or preserved from an existing tool
  - `writableDataSource.writeToColumnsThatMatch`
  - copy/move target parent and optional renamed destination
### 4.1) Input Gate (Required)
- Never infer destination branches or generator settings from prior examples.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context that has already been explicitly confirmed for this branch.
- Do not populate more than one destination branch inside the same `dataSourceColumn` object.
- Although the schema technically requires only `parent` for create, blank-shell create without `toolSettings.dataGenerators` is not validated in this repo. Default to configured create with at least one extract unless the user explicitly asks for a probe-style scaffold.
- String generator semantics confirmed for this repo:
  - `characterMap` is the set of valid random replacement characters.
  - each `&` in `pattern` resolves to one random character chosen from `characterMap`.
  - each `#` in `pattern` resolves to one random digit from `[0-9]`.
- If a string `pattern` uses `&`, require `characterMap` unless the user explicitly approves the tool's default character-map behavior.
- Do not invent number bounds/precision or date start/format/offset semantics that the user did not request.
- Do not call create until every extract has one resolved destination branch and one resolved generator branch.
### 4.2) Missing-Field Solicitation Contract
When required fields are missing, ask for all unresolved values in one prompt:
- Tool name (optional):
- For each extract:
  - extract name:
  - destination kind (`customColumn`, `suiteVariable`, or `writableDataSource`):
  - destination name/value:
  - generator type (`string`, `number`, or `date`):
  - string branch: `pattern`, `characterMap` when `&` is used:
  - number branch: `min`, `max`, optional `decimalPlaces`:
  - date branch: `startDate` mode/value, `outputDate` format/timeZone/locale as needed, optional `offset` fields:
Do not call create endpoints until the requested extract definitions are fully resolved.

### 4.3) Terminology Rule: "Data Source Column"
- In SOAtest/Virtualize, "data source column" is overloaded terminology: it may refer to a true datasource-backed column or to a tool-produced custom column such as `customColumn.customColumnName` emitted by a Data Bank or Data Generator.
- Downstream consumers that ask for a data source column may therefore validly bind to these tool-produced custom columns.
- The API and persisted local `.tst` may serialize the same destination differently; for example, API readback may show `customColumn.customColumnName`, while local YAML may materialize the corresponding writable-custom-column shape under `virtualDSCreator.writableColumns.customName`.
- Treat those forms as equivalent representations of the same downstream binding intent, not as conflicting models.

## 5) Preconditions
- API reachable and authenticated.
- Parent suite ids exist.
- If writable datasource targets are requested, the target datasource/column must already exist or be confirmed through discovery.
- If deterministic ordering is required, full root-child ordering can be read first.
## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile E for tool mutation steps,
   - use Skill 050 Profile D only if execution/traffic observation is later needed for validation.
1. Inspect current placement context:
   - `GET /v6/children?id=<suite-id>`
2. Resolve destination wiring before write:
   - if writable datasource columns are not already confirmed, inspect them first rather than guessing,
   - preserve existing top-level `dataSource` on update unless the user explicitly wants it changed.
3. Create Data Generator Tool(s):
   - `POST /v6/tools/dataGenerators` with:
     - `parent.id`
     - `name`
     - optional `dataSource`
     - `toolSettings.dataGenerators[]`
   - each extract must contain:
     - `name`
     - one `dataSourceColumn` branch
     - one `dataGenerator` branch keyed by `type`
4. If exact order is required among suite children:
   - compute full ordered child id list
   - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id` sequence
5. Read back created tool(s):
   - `GET /v6/tools/dataGenerators?id=<tool-id>`
6. Update tool configuration/name as needed:
   - `GET /v6/tools/dataGenerators?id=<tool-id>`
   - mutate only the intended fields/extracts
   - `PUT /v6/tools/dataGenerators?id=<tool-id>` using full read-merge-write payload
7. Copy and/or move tool when restructuring:
   - copy: `POST /v6/tools/copy` with `{ from.id, to.parent.id, to.name? }`
   - move: `POST /v6/tools/move` with `{ from.id, to.parent.id, to.name? }`
8. Delete temporary/obsolete tools:
   - `DELETE /v6/tools?id=<tool-id>`
## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md`.
- Build Data Generator payloads from user input + REST API schema + this card's field model.
- Do not copy YAML-internal class names such as `StringGenerator`, `NumberGenerator`, `DateTimeGenerator`, or `virtualDSCreator` into REST payloads; those are asset-serialization forms, not the `POST/PUT /v6/tools/dataGenerators` request shape.
## 6.0.1) Minimal Payload Shape Examples (Disambiguation Only)
This example is shape-only and must not be copied with literal values.
Rules:
- Replace all `{{...}}` placeholders from user-provided values and confirmed runtime intent.
- Use exactly one `dataSourceColumn` branch per extract.
- For updates, use `PUT /v6/tools/dataGenerators?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049.
- Preserve opaque readback fields such as top-level `dataSource` unless the user explicitly wants them changed.
Configured create shape:
```json
{
  "parent": {
    "id": "{{PARENT_SUITE_ID}}"
  },
  "name": "{{DATA_GENERATOR_TOOL_NAME}}",
  "toolSettings": {
    "dataGenerators": [
      {
        "name": "{{EXTRACT_NAME}}",
        "dataSourceColumn": {
          "suiteVariable": {
            "variable": "{{SUITE_VARIABLE_NAME}}"
          }
        },
        "dataGenerator": {
          "type": "string",
          "string": {
            "pattern": {
              "type": "fixed",
              "fixed": "Name-&&&&"
            },
            "characterMap": {
              "type": "fixed",
              "fixed": "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890"
            }
          }
        }
      }
    ]
  }
}
```
Destination branch variants:
- custom column:
  - `{"customColumn":{"customColumnName":"{{CUSTOM_COLUMN_NAME}}"}}`
- suite variable:
  - `{"suiteVariable":{"variable":"{{SUITE_VARIABLE_NAME}}"}}`
- writable datasource column:
  - `{"writableDataSource":{"writableDataSourceColumn":"{{WRITABLE_COLUMN_NAME}}","writeToColumnsThatMatch":"{{MATCH_RULE_OPTIONAL}}"}}`
Generator branch variants:
- number:
  - `{"type":"number","number":{"min":{"type":"fixed","fixed":"1"},"max":{"type":"fixed","fixed":"10"},"decimalPlaces":{"type":"fixed","fixed":"0"}}}`
- date:
  - `{"type":"date","date":{"startDate":{"type":"currentMidnight"},"outputDate":{"format":{"type":"fixed","fixed":"yyyy-MM-dd"},"timeZone":{"type":"fixed","fixed":"UTC"}},"offset":{"days":{"type":"fixed","fixed":"1"}}}}`
## 7) Validation
- Expected success pattern:
  - create/update/copy/move/read/delete calls return `200` on valid inputs.
- Post-condition checks:
  - tool id path reflects expected parent placement
  - readback shows the intended `toolSettings.dataGenerators[]` count and ordering
  - each extract readback preserves the intended destination branch (`customColumn`, `suiteVariable`, or `writableDataSource`)
  - each extract readback preserves the intended generator `type` and branch-specific settings
  - temporary copy/move artifacts are absent after cleanup

### 7.1) Validated Downstream Consumption Pattern
- Project history now includes a validated authoring path in `TestAssets/parasoftdemoapp/ParasoftDemoApp_QA_Experimental.tst`: a `Category Name Generator` Data Generator was inserted before `Create Category - POST`, emitted `CATEGORY_NAME` through `customColumn.customColumnName`, and the downstream REST request body consumed that value at the primitive `name` leaf with `${CATEGORY_NAME}`.
- API readback preserved the Data Generator config plus the literal-token REST body shape, while the persisted local `.tst` materialized the same flow as `virtualDSCreator.writableColumns.customName: CATEGORY_NAME` on the generator side and structured `formJson` leaf wiring (`mode: 3`, `columnName: CATEGORY_NAME`) on the REST body side.
- This validates primitive-leaf downstream request-body consumption from Data Generator output, even when the leaf lives inside a larger JSON object. It does not yet validate arbitrary complex-object or whole-subtree substitution.
- Runtime execution is not a required default step of this skill. Structural API/local readback remains sufficient unless the caller or owning workflow explicitly requests runtime verification.
- Project history note: the Parasoft Demo App category-name flow was manually runtime-validated successfully after authoring, confirming that the generated unique value was consumed by the downstream create call.

## 8) Failure Modes
- `400` create/update for invalid payload shape, multiple destination branches in one extract, or missing required `id`/parent.
- `404` when source/target ids do not exist.
- PUT default-reset risk if partial payload omits existing extracts or branch-specific settings.
- Destination mismatch risk when suite-variable names or writable-column names are guessed instead of confirmed.
- String-pattern mismatch risk when `pattern` uses `&` but `characterMap` is absent or unintentionally defaulted.
- Ambiguity risk if agent copies YAML serialization concepts directly into REST payloads instead of the API schema.
- Ordering drift if `PUT /v6/suites/children` payload omits required siblings.
## 9) Safety / Rollback
- Write skill (mutates `.tst` structure).
- Rollback options:
  - delete newly created tools
  - move copied tools back to original parent
  - restore previous child order via saved `children` snapshot
  - restore prior extract list via saved GET readback before PUT
## 10) Reuse Notes
- Primary target: SOAtest.
- Related endpoints:
  - `GET/PUT/DELETE /v6/tools`
  - `POST /v6/tools/copy`
  - `POST /v6/tools/move`
  - `POST/PUT/GET /v6/tools/dataGenerators`
  - `GET/PUT /v6/suites/children`
- Cross-cutting dependency for downstream consumers:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
- When destination discovery is unresolved, use `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md` before writing.
## 11) Example Configuration Pattern
Example string-generator extract semantics (example only, not implicit defaults):
- `pattern = Name-&&&&` means emit four random characters after `Name-`.
- `characterMap = ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890` supplies the allowed `&` replacement set.
- `pattern = ###-##-####` uses `#` for digit replacement without requiring `characterMap`.
