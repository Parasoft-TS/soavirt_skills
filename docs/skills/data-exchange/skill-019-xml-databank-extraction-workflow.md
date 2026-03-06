# Skill 019: XML Data Bank Extraction Lifecycle (Cross-Tool)

## 1) Skill Name
Add, modify, read, delete, copy, and validate XML Data Bank on XML-producing tool output

## 2) Objective
Create a reusable lifecycle pattern for chaining XML Data Banks to XML output, extracting XPath-selected values, and storing them in named data bank columns for downstream use.

## 3) Scope
- In scope:
  - create XML Data Bank via `POST /v6/tools/xmlDataBanks`
  - configure extraction mappings via `PUT /v6/tools/xmlDataBanks?id=...`
  - readback and verify configuration via `GET /v6/tools/xmlDataBanks?id=...`
  - delete XML Data Bank via `DELETE /v6/tools?id=...`
  - copy XML Data Bank via `POST /v6/tools/copy`
  - run focused execution and validate runtime stability via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - downstream variable-consumer tool authoring
  - business-logic assertions (covered by assertor skills)

## 4) Inputs
- Required:
  - target output-provider parent id that emits XML
  - XML Data Bank name
  - extraction list (`columnName` + XPath)
- Optional:
  - extraction options (`contentOnly`, missing/empty behavior)
  - explicit data source name if required by target context
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel emits XML payloads.
- Parent id resolves to an output-provider location that accepts chained tools.

## 6) Procedure
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - Construct `parent.id` from Skill 018 mapping; do NOT pass the producer tool id directly.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating the XML Data Bank.
  - do not guess or locally invent parent-path mappings.
2. Create XML Data Bank:
   - `POST /v6/tools/xmlDataBanks` with `parent.id` and `name`.
3. Configure extraction mappings:
   - `PUT /v6/tools/xmlDataBanks?id=<databank-id>`
   - set `toolSettings.selectedElements[]` entries:
     - `dataSourceColumn.customColumn.customColumnName`
     - `selectedElement.xpath`
     - `selectedElement.options.contentOnly` (for example `textContent`)
4. Read back tool and verify mappings persisted.
5. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied XML Data Bank via `GET /v6/tools/xmlDataBanks?id=<new-id>`.
6. Optional delete flow:
  - `DELETE /v6/tools?id=<databank-id>` and verify absence in descendants/children.
7. Execute focused run and confirm no parser/input-binding mismatch.

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Readback shows expected extraction mappings in `toolSettings.selectedElements[]`.
- Runtime remains stable and producer test executes successfully.

## 8) Failure Modes
- `400` invalid payload (for example unsupported `dataSource` value).
- Context-specific `dataSource` option mismatches (for example `New Datasource`) may fail with `400` where the accepted value is context-bound (for example `Untitled` in validated DB Tool flows).
- `404` invalid tool id or parent id.
- XPath extracts no value due to path mismatch against runtime XML structure.
- Input-binding mismatch if chained to non-semantic diagnostic output.
- copy placement/name collisions can cause unintended parent placement without immediate readback.

## 9) Safety / Rollback
- Write skill (adds/modifies chained tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<databank-id>`, or
  - restore previous config with saved GET snapshot.

## 10) Reuse Notes
- Applies to SOAtest: Yes (validated).
- Applies to Virtualize: not yet validated.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - Use GET -> mutate -> PUT for updates to existing XML Data Bank tools.
  - when new producer/output types are introduced, update Skill 018 first, then apply here.

## 11) Validation Snapshot (2026-03-03)
- Created under DB Tool semantic XML output:
  - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/DB Tool - Root Between Test And Child/Results as XML/XML Data Bank - DB Result Fields`
- Configured extractions:
  - `DB_ID` from `/results/resultSet/rows/row/ID/text()`
  - `DB_BALANCE` from `/results/resultSet/rows/row/BALANCE/text()`
- Runtime validation:
  - job `1633921850`
  - summary `failureCount=2`, `testRunCount=3` (expected setup failures only)
  - DB Tool test status `pass`
  - XML prolog parser error absent.
- Mirrored on child-suite DB Tool semantic XML output:
  - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/Child Test Suite/DB Tool - First Child Suite/Results as XML/XML Data Bank - Child DB Result Fields`
  - same extraction pattern (`DB_ID`, `DB_BALANCE`)
  - runtime validation:
    - job `318960184`
    - summary `failureCount=2`, `testRunCount=3`
    - child DB Tool test status `pass`
    - XML prolog parser error absent.
- Evidence folder:
  - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/`
