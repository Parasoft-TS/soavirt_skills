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
  - run focused execution and validate runtime stability via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - downstream variable-consumer tool authoring
  - business-logic assertions (covered by assertor skills)

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
1. Construct the output-provider parent path for the target producer tool:
  - REST Client: `<rest-client-id>/Response Traffic`
  - DB Tool: `<db-tool-id>/Results as XML`
  - Do NOT pass the producer tool id directly as `parent.id`; use the output-provider path.
  - See `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` Section 5 for canonical patterns.
2. Create JSON Data Bank:
   - `POST /v6/tools/jsonDataBanks` with `parent.id` and `name`.
3. Configure extraction mappings:
   - `PUT /v6/tools/jsonDataBanks?id=<databank-id>`
   - set selected-elements mappings for:
     - data bank column name
     - selected element XPath
4. Apply selector rule:
   - for JSON payloads, provide XPath-style selectors (Skill 011), not JSONPath.
5. Read back tool and verify mappings persisted.
6. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied JSON Data Bank via `GET /v6/tools/jsonDataBanks?id=<new-id>`.
7. Optional delete flow:
  - `DELETE /v6/tools?id=<databank-id>` and verify absence in descendants/children.
8. Execute focused run and confirm extraction is stable.

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
- Applies to SOAtest: Expected; endpoint/flow should be validated with first live run.
- Applies to Virtualize: not yet validated.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - Use GET -> mutate -> PUT for updates to existing JSON Data Bank tools.

## 11) Initial Validation Plan
1. Create under a REST Client JSON response output-provider anchor.
2. Add 1-2 XPath selectors for stable fields in known JSON response.
3. Run focused execution and verify tool readback and downstream variable availability.
