# TST Configuration Analysis Template

Use this template when producing Skill 052 output.

## A) Scope Resolution
- **Root suite:**
- **Resolved focus suite/path:**
- **Scope notes:** (for example setup-only, subtree-only, or full-file)

## B) Execution Context
- **Root/focus hierarchy (ordered):**
	- suite/test/tool nodes in YAML order
	- include setup placement and disabled-state markers where present
- **Loop and datasource binding context:**
	- list loop-related bindings and bound datasource names

## C) Step Flow
- **Ordered standalone steps in scope:**
	- **Test:** `<test name>`
	- **Consumes:** `${...}` entries only (or `none`)
	- **Produces:** producer symbols/columns only (or `none`)
	- **AppliesValidation:**
		- validation tool name/type
		- selector(s)
		- assertion/config summary

## D) Available Data Sources
- **Datasource records:**
	- name
	- family
	- columns

## Required Output Rules
- Section order is mandatory: `A`, `B`, `C`, `D`.
- Do not rename, omit, or reorder template sections in default mode.
- Preserve YAML order when listing suites/tests/tools/steps.
- Normalize all variable references to `${...}`.
- `Consumes` must include variables used by non-validation execution/configuration fields, including direct and transitive `${...}` environment-variable references.
- `Consumes` must exclude variables used only by assertor/validator definitions when those variables are already represented under `AppliesValidation`.
- De-duplicate and stable-sort `Consumes` and `Produces` lists.
- Use `Unknown (<reason>)` for unresolved required fields; do not omit fields.
