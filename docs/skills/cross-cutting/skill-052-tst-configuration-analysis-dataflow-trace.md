# Skill 052: TST Configuration Analysis and Dataflow Trace (YAML-Only)

## 1) Skill Name
TST configuration analysis and dataflow trace (read-only composite, YAML-only)

## 2) Objective
Produce four structured maps for a target `.tst`/suite using downloaded YAML as the source of truth:
- Execution Context
- Step Flow
- Available Data Sources

This skill is intended for questions like:
- "How are tests configured in this suite?"
- "Where does this dynamic value come from and where is it used?"
- "What validations are applied, and to which outputs/fields?"

## 3) Scope
- In scope:
  - analyze suite/test/tool hierarchy from YAML.
  - trace variable/dataflow using YAML fields only.
  - inventory validations from YAML assertor definitions.
  - resolve datasource family/columns from YAML datasource definitions.
- Out of scope:
  - any write mutation to tests/tools/datasources.
  - REST introspection endpoints other than file download.

## 4) Inputs
- Required:
  - `tstFileId`
- Optional:
  - `focusSuiteName` or `focusSuitePath` (YAML-resolved target scope)
  - `focusVariable`
  - `depth` (`brief`, `standard`, `deep`)

## 5) Dependencies
- `docs/skills/platform/skill-002-shared-file-transfer.md`
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
- `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`

## 6) Procedure
1. Retrieve YAML source of truth:
  - `GET /v6/files/download?id=<tstFileId>`
2. Parse structural baseline from YAML:
  - suite hierarchy (`TestSuite` nesting, child order, enabled/disabled state)
  - test/tool nodes and output-tool chains
  - resolve display naming for report output:
    - prefer the visible tool display name for tool-backed steps (for example `tool.name`),
    - use suite/test node name as fallback when no tool display name exists,
    - suppress internal/stored-name commentary in output.
3. Resolve datasource availability from YAML only:
  - detect datasource implementations (`TableDataSource`, `CSVDataSource`, and others present in YAML),
  - extract datasource names, identifiers, columns, and row/value availability where embedded
4. Build YAML-only variable/dataflow graph:
  - detect producers from YAML-defined producer tools and extension outputs,
  - detect consumers from parameterized fields (for example `parameterizedValue.column`, `columnName`, `${...}` placeholders),
  - apply recursive fallback consume detection for any tool type:
    - walk the entire tool/test YAML subtree and collect parameterization markers from any nested object path,
    - treat all discovered `columnName`, `parameterizedValue.column`, and `${...}` placeholders as consumed variables unless they are part of producer-definition blocks,
    - normalize consumed values to `${...}` in output.
  - resolve explicit datasource binding using `dataSourceNames` as authoritative source
5. Build validation inventory from YAML:
  - capture assertor/validator tools in scope,
  - extract assertion type, selector, operator/condition (where applicable), and expected source (fixed vs parameterized)
6. Emit map output with evidence labels:
  - `Execution Context`
  - `Step Flow`
  - `Available Data Sources`

## 6.1) YAML-Only Policy (Required)
- Do not call runtime tool/datasource introspection endpoints (`/v6/tools/*`, `/v6/datasources/*`, `/v6/descendants/assets`, `/v6/assets/data`) for this skill.
- Use REST only to download `.tst` file content.
- Treat YAML content as authoritative for:
  - datasource binding (`dataSourceNames`),
  - parameterization (`parameterizedValue`, `columnName`, `${...}`),
  - constrained REST details (`schemaURL`, `baseUrl`, constrained query/path parameters)

## 6.2) Output Contract (Required)
- `Execution Context` must include:
  - root/focus suite + ordered child hierarchy
  - setup/disabled placement where present
  - loop/data-source binding context where present
- `Step Flow` must include:
  - ordered standalone steps in scope
  - per step:
    - `consumes`: normalized `${...}` bullets only (including chained child tools except validation tools)
      - de-dup rule: if a variable is only referenced by chained validation tools, do not include it in `consumes`.
    - `produces`: only cross-step symbol/column producers from `JSON Data Bank`, `XML Data Bank`, `REST URL Data Bank`, `Text Data Bank`, `Data Generator Tool`, `Data Repository CRUD Tool`, `Extension Tool`; otherwise `none`
    - `appliesValidation`: chained validation tools with nested assertion details:
      - validation tool name/type
      - selector(s)
      - assertion-type/config summary
- `Available Data Sources` must include:
  - datasource id/name/family
  - columns (or `Unknown`)
  - fallback reason when unresolved

## 6.3) Missing-Data Rule
- Never omit a map because fields are missing.
- Use `Unknown` + explicit YAML reason (for example not present in serialized block).
- Confidence labels:
  - `confirmed` (explicit YAML field),
  - `inferred` (derived from nearby YAML linkage),
  - `partial` (incomplete YAML coverage for field)

## 7) Validation
- Ensure required maps are always present (`Execution Context`, `Step Flow`, `Available Data Sources`).
- Ensure all datasources in focus scope appear in `Available Data Sources`.
- Ensure all validation tools in focus scope appear under `Step Flow -> appliesValidation`.
- Ensure each reported consume/produce variable has YAML evidence anchor.

## 8) Failure Modes
- YAML variants across product versions may relocate or rename fields.
- Large files can make targeted focus resolution ambiguous without exact suite path.
- Some runtime-resolved values may not be serialized directly and require `inferred` labeling.

## 9) Safety / Rollback
- Read-only by default? Yes.
- No rollback required.

## 10) Reuse Notes
- Applies to SOAtest: Yes.
- Applies to Virtualize: Not validated.
- Preferred when REST payloads are sparse/minimal and YAML fidelity is higher.

## 11) Current Validation Status
- Migrated from prior mixed-source variant to YAML-only behavior.
- Runtime validation should be captured on representative suites.
