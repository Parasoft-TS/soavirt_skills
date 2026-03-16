# Skill 013: SOAtest Test Naming Policy (Deterministic Execution)

## 1) Skill Name
Deterministic test naming policy for SOAtest assets.

## 2) Objective
Prevent execution ambiguity by enforcing globally unique, stable test names and avoiding default tool/test names (for example `REST Client`).

The primary mitigation for focused verification in complex orchestrations is structural isolation such as Skill 074 disable-run-restore sandboxing. Naming remains important because `soatestOptions.testNames` is coarse and because clear names improve diagnostics, fallback targeting, and human inspection.

## 3) Why This Matters
- `soatestOptions.testNames` filtering is name-based and coarse.
- Duplicate names can cause multiple tests to run unintentionally.
- Unique naming enables deterministic targeted runs and cleaner reporting.
- In richer orchestrations, branch-aware naming helps humans and fallback execution understand whether a test belongs to baseline, happy-path lifecycle, negative, or security coverage.

## 4) Scope
- In scope:
  - naming rules for new test and tool nodes
  - rename guidance for legacy default names
  - preflight checks before creating tests
  - caller-supplied branch-aware naming profile for more structured orchestrations
- Out of scope:
  - business-domain naming conventions beyond execution determinism
  - chained tools do not need to adhere to the deterministic naming policy; default names are acceptable unless a calling workflow has a stronger readability reason

## 5) Naming Rules
1. Never keep default generated names (examples: `REST Client`, `DB Tool`, `JSON Assertor`) for tools whose parent is a test suite.
2. Every executable test name should be unique across the workspace.
3. Use stable names that do not depend on row order, temporary data, or same-run ids.
4. Prefer explicit intent in name text (operation/endpoint/purpose).
5. When the calling workflow already models branch families or cluster structure, encode that role explicitly rather than hiding all tests behind flat operation-only names.

## 6) Recommended Patterns
### 6.1 General deterministic pattern
Use a deterministic format that naturally avoids collisions:

`<SuiteContext> :: <OperationOrEndpoint> :: <Expectation>`

Example:

`Root Smoke :: GET /customers/{customerId}/accounts :: 200 and account list`

### 6.2 Branch-aware extension for richer orchestrations
When the calling workflow has stable cluster and branch-family structure, extend the generic pattern instead of inventing ad hoc local rules:

`<Cluster> :: <BranchFamily> :: <ScenarioOrOperation> :: <CaseOrExpectation>`

Representative examples:
- `Categories :: Happy Path :: Baseline Snapshot :: GET /categories returns baseline collection`
- `Categories :: Happy Path :: Lifecycle Flow :: POST create category succeeds`
- `Categories :: Negative Coverage :: PUT /categories/{id} :: invalid id returns 404`
- `Categories :: Security Coverage :: DELETE /categories/{id} :: pen test branch`

Guidance:
- `Cluster` should be stable across the authored subtree.
- `BranchFamily` should use durable execution-scope labels such as `Happy Path`, `Optional / Deferred`, `Negative Coverage`, or `Security Coverage`.
- `ScenarioOrOperation` may be either the scenario name or the anchored operation, whichever better preserves deterministic meaning.
- `CaseOrExpectation` should describe the step, case, or assertion intent in human-readable form.

## 7) Procedure (When Creating New Tests)
1. Preflight existing names via `GET /v6/descendants/assets?id=<file-id>`.
2. Build a candidate name using either the general deterministic pattern or the caller-supplied branch-aware extension.
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
- In orchestrations that can isolate execution structurally, prefer structural isolation first and naming-based execution targeting second.

## 11) Reuse Notes
- Primary target: SOAtest.
- Virtualize naming hygiene may still be useful even when the specific execution-selection constraint differs.
- Typical rich caller: Skill 065 or other orchestrations that need stable branch-aware naming without forking this policy into a custom naming system.
