# Skill Family: TST Object Manipulation
Purpose: route object-level mutation across `.tst` suites, tools, data sources, environments, and related nodes without defaulting to ad hoc local YAML edits.
## Scope
This family covers structural or identity-oriented operations on these object classes:
- test suites
- responder suites
- tools
- data sources
- environments
- global properties
It is the routing surface for choosing the right validated owner, not a single endpoint card.
## Ownership Model
Use the smallest validated owner for the requested mutation class:
- in-place rename of supported suites/tools:
  - `docs/skills/structure/skill-068-rename-object.md`
- pure tool copy / move / delete:
  - `docs/skills/structure/skill-family-tool-lifecycle-operations.md`
- disabled-state mutation for supported asset kinds:
  - `docs/skills/structure/skill-family-disabled-state-mutation.md`
- broad test-suite lifecycle and configuration:
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
- datasource move with implementation-type ambiguity:
  - `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
- SOAtest environment lifecycle mechanics:
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
## Design Rule
- Keep one umbrella family card for intent routing.
- Prefer the smallest validated API-backed owner for each mutation class.
- Use local YAML only when a validated owning leaf explicitly names a YAML exception and routes rollback through Skill 006.
- Do not use YAML helper cards as a substitute for ordinary rename, copy, move, delete, enable/disable, or routine configuration work when a supported API path exists.
## Selection Guide
- Need to rename an existing suite or supported tool in place?
  - use `docs/skills/structure/skill-068-rename-object.md`
- Need to copy, move, or delete an existing tool and nothing broader is changing?
  - use `docs/skills/structure/skill-family-tool-lifecycle-operations.md`
- Need to enable or disable an existing supported asset without broader configuration changes?
  - use `docs/skills/structure/skill-family-disabled-state-mutation.md`
- Need to create, configure, copy, move, reorder, or otherwise manage a suite as a lifecycle object?
  - use `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
- Need to move exactly one datasource implementation type when the API branch may not target it deterministically?
  - use `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
- Need to manage internal or referenced SOAtest environment nodes?
  - use `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
## Dependency Model
- Atomic operation cards should keep endpoint, payload, and verification logic local.
- Family cards should route to the owning atomic card rather than duplicating endpoint details.
- YAML-authorized exception cards should depend on Skill 006 and explicitly describe why API-backed ownership is insufficient for that branch.
## Validation Standard
Every atomic manipulation card must include:
1. explicit target-id or local-target resolution rules
2. exact endpoint and payload contract
3. before/after evidence capture
4. deterministic post-condition verification
5. fail-closed handling when target class, parent compatibility, or readback authority is unclear
## Known Constraints
- **Environment child ordering is constrained**:
  - `PUT /v6/suites/children` does not allow arbitrary `QA`/`DEV` swaps in this target file.
  - Even with explicit `DEV` then `QA` payload, persisted order remained `QA` then `DEV`.
  - Treat environment reordering as constrained/not supported unless a future product/API version proves otherwise.
- **Data source move can be implementation-ambiguous for shared display ids**:
  - `POST /v6/datasources/move` may move one underlying implementation per call when multiple data source implementations are represented under the same visible node id/path.
  - Validation must use before/after YAML (implementation type traces) in addition to REST child listings.
  - For requests like `move TableDataSource` / `move FileDataSource` / `move WritableDataSource`, confirm target implementation type explicitly after move.
## Current Boundary Notes
- The generic tools API is the right owner for tool copy/move/delete and disabled-state writes, but not for rename.
- Representative class-specific tool PUT schemas show `name` remains the primary universal tool property that still requires class-specific update ownership.
- Test-node rename and direct test-node disabled-state readback remain future extension areas unless a later card codifies them explicitly.
