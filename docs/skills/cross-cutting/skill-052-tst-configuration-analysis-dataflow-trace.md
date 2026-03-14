# Skill 052: TST Configuration Analysis and Dataflow Trace (YAML-Only)

## 1) Skill Name
TST configuration analysis and dataflow trace (read-only composite, YAML-only)

## 2) Objective
Produce three structured maps for a target `.tst`/suite using downloaded YAML as the source of truth:
- Execution Context
- Step Flow
- Available Data Sources

Validation details are represented in `Step Flow -> AppliesValidation`.

This skill is intended for questions like:
- "How are tests configured in this suite?"
- "Where does this dynamic value come from and where is it used?"
- "What validations are applied, and to which outputs/fields?"
- "Can you analyze this test suite for me?"

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
- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/platform/skill-002-shared-file-transfer.md`
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
- `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`

## 6) Procedure
0. Phase A - target resolution (if target identity/scope is unresolved):
  - allowed endpoints: `GET /v6/children`, `GET /v6/descendants/files`
  - use only to resolve `tstFileId` and optional suite path/name scope.
  - once `tstFileId` is resolved, transition to Phase B.
1. Phase B - retrieve YAML source of truth:
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
  - detect consumers from non-validation execution/configuration fields (for example endpoint/router/baseUrl/schema/wsdl/query/path/body/connection and SQL fields),
  - apply step-attribution boundary before emitting `Consumes`:
    - for a tool-backed step, `Consumes` must be attributed only from the step's primary executable node and other non-validation execution/configuration fields owned by that step,
    - do not attribute variables from chained assertor/validator child-tool configuration to the parent step's `Consumes`,
    - treat chained assertor/validator child-tool references as validation scope and emit them under `AppliesValidation`,
  - include environment-variable consumes from `${...}` references in execution/configuration fields.
    - include direct references in the current step (for example router/baseUrl/schema/wsdl fields),
    - include transitive references when the step binds to shared configuration by name (for example DB connection properties referenced through `propertyName`),
  - exclude validation-only variable references from `Consumes` when the variable appears only inside assertor/validator definitions and is already represented in `AppliesValidation`,
  - apply recursive fallback consume detection for any tool type:
    - walk the entire tool/test YAML subtree and collect parameterization markers from any nested object path,
    - treat discovered `columnName`, `parameterizedValue.column`, and `${...}` placeholders as consumed variables only when they are not in producer-definition blocks and not validation-only fields,
    - normalize consumed values to `${...}` in output.
  - apply validation-reference precedence before finalizing `Consumes`:
    - if a variable is discovered only inside chained assertor/validator fields, emit it only under `AppliesValidation` and never under `Consumes`, even if recursive subtree scanning found it first,
    - only emit a variable in both `Consumes` and `AppliesValidation` when YAML evidence shows it is used by both non-validation execution/configuration fields and validation configuration fields in the same scoped step,
  - resolve explicit datasource binding using `dataSourceNames` as authoritative source
5. Build validation inventory from YAML:
  - capture assertor/validator tools in scope,
  - extract assertion type, selector, operator/condition (where applicable), and expected source (fixed vs parameterized)
6. Emit output using:
  - `docs/templates/tst-configuration-analysis-template.md`
  - include all required sections and fields in template order.
7. Fail-closed enforcement:
  - if any forbidden post-download endpoint is called (see 6.1), abort and restart Phase B using YAML evidence only.

## 6.1) YAML-Only Policy (Required)
- Do not call runtime tool/datasource introspection endpoints (`/v6/tools/*`, `/v6/datasources/*`, `/v6/descendants/assets`, `/v6/assets/data`) for this skill.
- Use REST only to download `.tst` file content.
- Treat YAML content as authoritative for:
  - datasource binding (`dataSourceNames`),
  - parameterization (`parameterizedValue`, `columnName`, `${...}`),
  - constrained REST details (`schemaURL`, `baseUrl`, constrained query/path parameters)
- Explicit boundary rule:
  - before YAML download: only target-resolution reads are allowed (`/v6/children`, `/v6/descendants/files`) when needed to resolve target id/scope.
  - after YAML download begins: no additional API discovery/introspection calls are allowed for analysis evidence.
- Forbidden post-download endpoints (non-exhaustive):
  - `/v6/children`
  - `/v6/descendants/files`
  - `/v6/descendants/assets`
  - `/v6/assets/data`
  - `/v6/tools/*`
  - `/v6/datasources/*`

## 6.2) Template Compliance (Required)
- Use `docs/templates/tst-configuration-analysis-template.md` as the canonical output contract.
- The template owns:
  - section names and order,
  - required fields per section,
  - normalization rules,
  - missing-data fallback and confidence-label conventions.
- Do not duplicate or override template-owned output-structure rules in this skill card.
- Response validity gate:
  - output is invalid unless template sections appear in exact template order.
  - freeform/non-template summaries are not allowed for default Skill 052 output.
  - required fields must be present; unresolved fields must be emitted as `Unknown (<reason>)`.

## 6.3) Common Divergence Traps (Required Handling)
- Trap: continuing API asset introspection after target resolution.
  - Required handling: stop API exploration and continue with YAML-only evidence.
- Trap: searching repo-local snapshots or `work/` artifacts before Phase A when the prompt names a runtime `.tst`.
  - Required handling: resolve `tstFileId` via `GET /v6/children` / `GET /v6/descendants/files`, then download via `GET /v6/files/download?id=<tstFileId>`, then continue with YAML-only evidence.
- Trap: returning narrative summary instead of template contract.
  - Required handling: regenerate output in exact template structure.
- Trap: mixing discovery evidence and analysis evidence without source distinction.
  - Required handling: use API reads only for target resolution; all analysis claims must be YAML-evidenced.
- Trap: populating `Consumes` with variables that are only used by assertor/validator tools.
  - Required handling: move those references to `AppliesValidation` and exclude them from `Consumes` unless they are also used by non-validation execution/configuration fields.
- Trap: recursive subtree scanning finds a validation-child variable before step ownership is resolved.
  - Required handling: apply the step-attribution boundary first, then apply validation-reference precedence so child assertor/validator references cannot be emitted as parent-step `Consumes`.
- Trap: missing environment-variable consumes in endpoint/baseUrl/connection configuration.
  - Required handling: extract `${...}` references from execution/configuration fields, including transitive shared-property bindings (for example named DB property references).

## 7) Validation
- Ensure required maps are always present (`Execution Context`, `Step Flow`, `Available Data Sources`).
- Ensure output complies with `docs/templates/tst-configuration-analysis-template.md`.
- Ensure all datasources in focus scope appear in `Available Data Sources`.
- Ensure all validation tools in focus scope appear under `Step Flow -> AppliesValidation`.
- Ensure `Step Flow -> Consumes` excludes validation-only variable references that are already represented in `AppliesValidation`.
- Ensure `Step Flow -> Consumes` for a tool-backed step is attributed from the primary executable node's non-validation execution/configuration fields, not from chained assertor/validator child-tool configuration.
- Ensure `Step Flow -> Consumes` includes environment-variable references used by execution/configuration fields, including transitive shared-property bindings.
- Ensure each reported consume/produce variable has YAML evidence anchor.
- Mandatory pre-response compliance checklist:
  1. `tstFileId` and optional focus scope resolved.
  2. YAML downloaded via `/v6/files/download`.
  3. No forbidden post-download API calls were used for analysis evidence.
  4. Template sections/fields are present in exact order.
  5. Required maps are present (`Execution Context`, `Step Flow`, `Available Data Sources`).
  6. Claims about flows/variables/validations/datasources include YAML evidence anchors.
  7. `Consumes` excludes validation-only references, respects step-attribution boundary, and includes direct/transitive environment-variable consumes.

## 8) Failure Modes
- YAML variants across product versions may relocate or rename fields.
- Large files can make targeted focus resolution ambiguous without exact suite path.
- Some runtime-resolved values may not be serialized directly and require `inferred` labeling.

## 9) Safety / Rollback
- Read-only by default? Yes.
- No rollback required.

## 10) Reuse Notes
- Primary target: SOAtest.
- Not applicable in Virtualize.
- Preferred when REST payloads are sparse/minimal and YAML fidelity is higher.
