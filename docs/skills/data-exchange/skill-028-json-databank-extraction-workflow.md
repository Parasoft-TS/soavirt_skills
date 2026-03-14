# Skill 028: JSON Data Bank Extraction Lifecycle (Cross-Tool)

## 1) Skill Name
Add, modify, read, delete, copy, and validate JSON Data Bank on JSON-producing tool output

## 2) Objective
Create a reusable lifecycle pattern for chaining JSON Data Banks to JSON output, extracting XPath-selected values, and storing them in named data bank columns for downstream use.

This serves the same data-exchange purpose as XML Data Bank, but for JSON payloads.

## 3) Scope
- In scope:
  - create JSON Data Bank via `POST /v6/tools/jsonDataBanks`
  - configure extraction mappings via `PUT /v6/tools/jsonDataBanks?id=...`
  - readback and verify configuration via `GET /v6/tools/jsonDataBanks?id=...`
  - delete JSON Data Bank via `DELETE /v6/tools?id=...`
  - copy JSON Data Bank via `POST /v6/tools/copy`
  - run focused execution and validate runtime stability by following the Skill 012 execution-triad contract
- Out of scope:
  - downstream variable-consumer tool authoring
  - business-logic assertions (covered by assertor skills)

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
  - target output-provider parent id that emits JSON
  - JSON Data Bank name
  - extraction list (`columnName` + XPath)
- Optional:
  - extraction options (missing/empty behavior)
  - explicit data source name if required by target context
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits JSON payloads.
- Parent id resolves to an output-provider location that accepts chained tools.
- JSON selector rule must be loaded:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - use XPath-style selectors for JSON fields, not JSONPath.

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - Construct `parent.id` from Skill 018 mapping; do NOT pass the producer tool id directly.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating the JSON Data Bank.
  - do not guess or locally invent parent-path mappings.
2. Create JSON Data Bank:
   - `POST /v6/tools/jsonDataBanks` with `parent.id` and `name`.
3. Configure extraction mappings:
   - `PUT /v6/tools/jsonDataBanks?id=<databank-id>`
   - set selected-elements mappings for:
     - data bank column name
     - selected element XPath
4. Apply selector rule:
   - for JSON payloads, provide XPath-style selectors (Skill 011), not JSONPath.
  - Skill 054 boundary: XML text-node normalization (`/text()`) is not required for JSON selector fields; keep JSON selectors in XPath-over-JSON form.
5. Read back tool and verify mappings persisted.
6. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied JSON Data Bank via `GET /v6/tools/jsonDataBanks?id=<new-id>`.
7. Optional delete flow:
  - `DELETE /v6/tools?id=<databank-id>` and verify absence in descendants/children.
8. Execute focused run by following the Skill 012 execution-triad contract and confirm extraction is stable.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in `docs/workflow/agent-workflow.md` (Decision Rule).
- Construct JSON Data Bank payloads from REST API schema + skill semantics.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Before using it, follow:
- Section 5) Preconditions
- Section 6) Procedure

Rules:
- Replace all `{{...}}` placeholders from runtime evidence + user intent.
- Do not reuse sample ids/names/xpaths/column names unless explicitly requested.
- For extraction options not shown here, keep the same envelope and use OpenAPI schema for JSON Data Bank tool settings.
- For updates, use `PUT /v6/tools/jsonDataBanks?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049 (same `toolSettings` shape, no `parent` field).

```json
{
  "parent": {
    "id": "{{PARENT_OUTPUT_PROVIDER_ID}}"
  },
  "name": "{{JSON_DATABANK_NAME}}",
  "toolSettings": {
    "selectedElements": [
      {
        "dataSourceColumn": {
          "customColumn": {
            "customColumnName": "{{COLUMN_NAME}}"
          }
        },
        "selectedElement": {
          "xpath": "{{XPATH_FROM_OBSERVED_JSON}}",
          "options": {
            "contentOnly": "textContent"
          }
        }
      }
    ]
  }
}
```

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback shows expected extraction mappings persisted on the tool.
- Runtime remains stable and producer test executes successfully.
- Extracted values resolve in downstream variables where expected.

## 8) Failure Modes
- `400` invalid payload shape or unsupported field values.
- `404` invalid tool id or parent id.
- XPath extracts no value due to path mismatch against runtime JSON structure.
- Selector syntax mismatch when JSONPath is used in XPath-expected fields.
- Input-binding mismatch if chained to non-semantic diagnostic output.
- copy placement/name collisions can cause unintended parent placement without immediate readback.

## 9) Safety / Rollback
- Write skill (adds/modifies chained tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<databank-id>`, or
  - restore previous config with saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
