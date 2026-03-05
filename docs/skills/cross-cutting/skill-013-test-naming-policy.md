# Skill 013: SOAtest Test Naming Policy (Deterministic Execution)

## 1) Skill Name
Deterministic test naming policy for SOAtest assets.

## 2) Objective
Prevent execution ambiguity by enforcing globally unique, stable test names and avoiding default tool/test names (for example `REST Client`).

## 3) Why This Matters
- `soatestOptions.testNames` filtering is name-based and coarse.
- Duplicate names can cause multiple tests to run unintentionally.
- Unique naming enables deterministic targeted runs and cleaner reporting.

## 4) Scope
- In scope:
  - naming rules for new test and tool nodes
  - rename guidance for legacy default names
  - preflight checks before creating tests
- Out of scope:
  - business-domain naming conventions beyond execution determinism
  - chained tools do not need to adhere to the deterministic naming policy, the default names are acceptable

## 5) Naming Rules
1. Never keep default generated names (examples: `REST Client`, `DB Tool`, `JSON Assertor`) for tools whose parent is a test suite.
2. Every executable test name should be unique across the workspace.
3. Use stable names that do not depend on row order or temporary data.
4. Prefer explicit intent in name text (operation/endpoint/purpose).

## 6) Recommended Pattern
Use a deterministic format that naturally avoids collisions:

`<SuiteContext> :: <OperationOrEndpoint> :: <Expectation>`

Example:

`Root Smoke :: GET /customers/{customerId}/accounts :: 200 and account list`

## 7) Procedure (When Creating New Tests)
1. Preflight existing names via `GET /v6/descendants/assets?id=<file-id>`.
2. Build candidate name using the recommended pattern.
3. Check candidate uniqueness against current descendants.
4. Create test/tool with explicit name (never default name).
5. Post-verify by reading descendants and confirming a single exact-name match.

## 8) Procedure (When Retrofitting Existing Assets)
1. Discover duplicate/default names in descendants.
2. Rename to deterministic unique names using class-specific update endpoints.
3. Re-run name scan until duplicates are removed.

## 9) Validation
- For each target name:
  - exact-match count in descendants should be `1`.
- For `testNames`-based execution:
  - targeted run should execute intended named test(s) without same-name collisions.

## 10) Known Constraint Linkage
- This policy is required due to validated behavior in:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `soatestOptions.testNames` being coarse name-based selection.

## 11) Reuse Notes
- Applies to SOAtest: Yes.
- Applies to Virtualize: partially relevant (naming hygiene still useful), execution-selection behavior no applicable.
