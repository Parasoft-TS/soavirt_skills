# Skill 052: TST Configuration Analysis and Dataflow Trace (YAML-Only)
## 1) Skill Name
TST configuration analysis and dataflow trace (read-only composite, local YAML-only)
## 2) Objective
Produce three structured maps for a target `.tst`/suite using local YAML as the source of truth:
- Execution Context
- Step Flow
- Available Data Sources
Validation details are represented in `Step Flow -> AppliesValidation`.
## 3) Scope
- In scope:
  - analyze suite/test/tool hierarchy from local YAML
  - trace variable/dataflow using YAML fields only
  - inventory validations from YAML assertor definitions
  - resolve datasource family/columns from YAML datasource definitions
- Out of scope:
  - any write mutation to tests/tools/datasources
  - runtime/API introspection as analysis evidence after local file read begins
## 4) Inputs
- Required:
  - explicit local `.tst` path, or
  - enough file-backed identity to resolve one local `.tst` through Skill 001
- Optional:
  - `focusSuiteName` or `focusSuitePath`
  - `focusVariable`
  - `depth` (`brief`, `standard`, `deep`)
  - active project context
## 5) Dependencies
- Required when target/scope is unresolved:
  - `docs/skills/platform/skill-001-shared-introspection.md`
- Required:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
## 6) Procedure
0. Resolve the local `.tst` target and optional focus scope:
  - if an explicit local path is provided, use it directly
  - otherwise resolve one local `.tst` through Skill 001 using active-project-first and likely-root-first search
  - if the focus suite path/name remains ambiguous after local YAML inspection, ask the user to clarify rather than guessing
1. Read the local `.tst` YAML.
2. Parse the structural baseline from YAML:
  - suite hierarchy (`TestSuite` nesting, child order, enabled/disabled state)
  - test/tool nodes and output-tool chains
  - resolve display naming for report output:
    - prefer the visible tool display name for tool-backed steps
    - use suite/test node name as fallback when no tool display name exists
3. Resolve datasource availability from YAML only:
  - detect datasource implementations (`TableDataSource`, `CSVDataSource`, and others present in YAML)
  - extract datasource names, identifiers, columns, and row/value availability where embedded
4. Build the YAML-only variable/dataflow graph:
  - detect producers from YAML-defined producer tools and extension outputs
  - detect consumers from non-validation execution/configuration fields
  - apply step-attribution boundary before emitting `Consumes`
  - include environment-variable consumes from `${...}` references in execution/configuration fields
  - exclude validation-only variable references from `Consumes` when the variable appears only inside assertor/validator definitions and is already represented in `AppliesValidation`
  - apply recursive fallback consume detection for any tool type, but only after ownership and validation-boundary checks are applied
  - resolve explicit datasource binding using `dataSourceNames` as the authoritative source
5. Build validation inventory from YAML:
  - capture assertor/validator tools in scope
  - extract assertion type, selector, operator/condition, and expected source where applicable
6. Emit output using `docs/templates/tst-configuration-analysis-template.md`.
7. Fail-closed enforcement:
  - if analysis claims depend on anything other than local YAML evidence, restart using only the allowed evidence base
## 6.1) Local YAML-Only Policy (Required)
- Do not use runtime tool/datasource introspection as analysis evidence.
- Treat local YAML content as authoritative for:
  - datasource binding (`dataSourceNames`)
  - parameterization (`parameterizedValue`, `columnName`, `${...}`)
  - constrained REST details (`schemaURL`, `baseUrl`, constrained query/path parameters)
- Explicit boundary rule:
  - before local file read: only local path-resolution steps are allowed when needed to resolve target id/scope
  - after local file read begins: no additional API/runtime discovery is allowed for analysis evidence
## 6.2) Template Compliance (Required)
- Use `docs/templates/tst-configuration-analysis-template.md` as the canonical output contract.
- The template owns:
  - section names and order
  - required fields per section
  - normalization rules
  - missing-data fallback and confidence-label conventions
- Response validity gate:
  - output is invalid unless template sections appear in exact order
  - freeform/non-template summaries are not allowed for default Skill 052 output
  - unresolved required fields must be emitted as `Unknown (<reason>)`
## 6.3) Common Divergence Traps (Required Handling)
- Trap: continuing runtime/API introspection after local file read.
  - Required handling: stop runtime/API exploration and continue with local YAML-only evidence.
- Trap: searching transient `work/` artifacts before resolving the actual local `.tst` target.
  - Required handling: resolve the `.tst` locally first, then continue with YAML-only evidence.
- Trap: returning a narrative summary instead of the template contract.
  - Required handling: regenerate output in exact template structure.
- Trap: populating `Consumes` with variables that are only used by assertor/validator tools.
  - Required handling: move those references to `AppliesValidation` unless YAML shows both validation and non-validation use.
## 7) Validation
- Ensure required maps are always present (`Execution Context`, `Step Flow`, `Available Data Sources`).
- Ensure output complies with `docs/templates/tst-configuration-analysis-template.md`.
- Ensure all datasources in focus scope appear in `Available Data Sources`.
- Ensure all validation tools in focus scope appear under `Step Flow -> AppliesValidation`.
- Ensure each reported consume/produce variable has local YAML evidence.
- Mandatory pre-response compliance checklist:
  1. local `.tst` target and optional focus scope resolved
  2. local YAML read successfully
  3. no runtime/API discovery was used as analysis evidence after local file read began
  4. template sections/fields are present in exact order
  5. claims about flows/variables/validations/datasources include local YAML evidence anchors
  6. `Consumes` excludes validation-only references and respects step-attribution boundary
## 8) Failure Modes
- YAML variants across product versions may relocate or rename fields.
- Large files can make targeted focus resolution ambiguous without an exact suite path.
- Some runtime-resolved values may not be serialized directly and require `inferred` labeling.
## 9) Safety / Rollback
- Read-only by default? Yes
- No rollback required.
## 10) Reuse Notes
- Preferred when local `.tst` fidelity is higher than sparse runtime payloads.
- This card is intentionally local/YAML-only once the target file is resolved.
