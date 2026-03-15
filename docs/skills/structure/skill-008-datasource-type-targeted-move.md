# Skill 008: Data Source Type-Targeted Move
## 1) Skill Name
Data source type-targeted move (suite-to-suite, hybrid local+API)
## 2) Objective
Move exactly one selected data source implementation type from one suite to another inside one `.tst`, without moving sibling datasource types, while using the local `.tst` path as the authoritative asset anchor and the localhost API only where runtime object ids or move semantics are required.
## 3) Scope
- In scope:
  - SOAtest `.tst` suite-to-suite datasource moves within one local file-backed asset
  - type-targeted moves for `TableDataSource`, `FileDataSource`, `WritableDataSource`, `CSVDataSource`, `DbDataSource`, `ExcelDataSource`
  - hybrid execution:
    - local path/YAML evidence for file-backed targeting and type confirmation
    - API move path when exact runtime datasource and suite ids can be resolved safely
    - local YAML fallback when the API branch cannot target the intended datasource implementation deterministically
- Out of scope:
  - cross-file moves between different `.tst` assets in one step
  - semantic edits to datasource internals beyond relocation
## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
- Required for local YAML fallback:
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
## 4) Inputs
- Required:
  - local `.tst` path or enough file-backed identity to resolve it through Skill 001
  - source suite reference
  - destination suite reference
  - target implementation type (for example `TableDataSource`)
- Optional:
  - target datasource name in destination suite
  - active project context
## 5) Preconditions
- The local `.tst` is readable.
- If the API move branch is used, API auth/base path is valid through Skill 050.
- A rollback-capable local edit path exists before local YAML fallback begins.
## 6) Procedure
1. Resolve the local `.tst` path through Skill 001.
2. Inspect local YAML first to confirm:
   - the source and destination suites exist in the file-backed asset
   - the target datasource implementation type exists in the source suite
   - sibling datasource types that must remain in place are visible
3. Decide the branch:
   - if one exact runtime datasource candidate of the requested implementation type and the destination suite runtime id can be resolved safely, enter the API branch
   - otherwise use the local YAML fallback branch
4. API branch:
   - enter Skill 050 before runtime-object discovery or mutation
   - resolve source datasource and destination suite runtime ids
   - if several runtime datasource candidates remain or type identity is still ambiguous, fail closed and switch to the local YAML fallback branch rather than guessing
   - call the supported datasource move route
5. Verify the API branch with both runtime and local evidence:
   - API readback confirms the runtime move landed under the intended destination suite
   - local file readback confirms the watcher-visible `.tst` now reflects the intended type move
   - if the wrong implementation moved, stop and recover through the local YAML fallback branch rather than treating the API result as success
6. Local YAML fallback branch:
   - use Skill 006 to create a rollback-preserving local edit envelope
   - move only the datasource block where `impl.$type == <target-type>` from the source suite to the destination suite
   - do not broaden the edit to sibling datasource types or unrelated suite structure
7. Re-read the local `.tst` and confirm postconditions.
## 7) Validation
- The source suite no longer contains the target implementation type.
- The destination suite now contains the target implementation type.
- Non-target sibling datasource types remain in the source suite.
- If the API branch was used, runtime readback and local file readback agree on the final location.
## 8) Failure Modes
- Multiple datasource runtime ids map ambiguously to one generic datasource family and the card guesses.
- The API move branch moves a different implementation first and that outcome is accepted without local verification.
- The local YAML fallback broadens beyond the intended datasource block.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback plan for writes:
  - for the API branch, verify immediately and stop on wrong-target movement
  - for the local YAML fallback branch, use the rollback-preserving workflow in Skill 006
## 10) Reuse Notes
- This is a hybrid card: local `.tst` identity and type evidence come first, while runtime ids and the move call remain API-mediated.
- This card is a narrow YAML-authorized exception and must not be used for ordinary tool/suite rename, generic tool copy/move/delete, disabled-state mutation, or routine configuration edits when supported API owners exist.
- Use Skill 051 before mutation when datasource-family or column identity is still unclear.
