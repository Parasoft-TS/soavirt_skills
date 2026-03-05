# Skill Card 007: TST Content Summarization and Intent Mapping

## 1) Skill Name
TST Content Summarization (REST + YAML Correlation)

## 2) Objective
- Produce an accurate, user-friendly summary of what a `.tst` file contains and what it is doing.
- Build a translation layer between user intent language and concrete editable targets in the TST.
- Highlight the application/service interface under test and the currently active environment.

## 3) Layer
- L1 Asset Operations

## 4) Dependencies
- `skill-001-shared-introspection.md` (find/select target `.tst`)
- `skill-002-shared-file-transfer.md` (download current YAML safely)
- Optional: `skill-006-safe-file-refactor-composite.md` for safe pre-edit copy workflows

## 5) Inputs
- Required:
  - `tstFileId`
- Optional:
  - `audience` (`technical`, `mixed`, `non-technical`)
  - `focusAreas` (for example `environments`, `data sources`, `assertions`, `chained tests`)

## 6) Procedure
1. Query REST structure with `GET /v6/descendants/assets?id=<tstFileId>`.
2. Query searchable field roll-up with `POST /v6/assets/data` using `scope.assetIds`.
3. Download YAML via `GET /v6/files/download?id=<tstFileId>`.
4. Correlate REST nodes to YAML sections:
   - REST hierarchy (`children`, `type`) ↔ YAML suite/test/tool nesting.
   - REST names/ids ↔ YAML `name`, `testID`, and nested tool names.
   - `assets/data` fields ↔ user-facing config highlights (URLs, path params, assertions).
5. If YAML indicates `EnvironmentReference`, resolve external dependency:
  - find referenced `.env` or `.envs` file on server
  - download dependency file
  - parse XML and resolve active environment
    - `.env`: single environment node
    - `.envs`: multiple environment nodes; select by `activeEnvironmentName`
  - enrich Section B with active environment variables from external file
5. Produce three outputs:
   - Structural summary (what exists)
   - Behavioral summary (what it appears to do)
   - Refactor target map (where common requested edits would apply)

## 7) Output Contract
- Include:
  - title format: `TST Summary - <filename.tst>`
  - application/service interface under test
  - root suite name and top-level children
  - currently active environment
  - active environment variable name/value table
  - suite/test hierarchy (default view, governed by size heuristic)
  - optional deep view for data sources and chained tools
  - business flow summary named by suite
- Use `docs/templates/tst-human-friendly-summary-template.md` as the default response shape.
- Include protocol/transport deep detail only when it helps answer the user’s question.
- Exclude risk/opinion/recommendation sections by default unless user explicitly asks.
- In Section B, include source context (for example referenced `.envs` filename) but omit implementation-detail narration (for example "downloaded/parsing" mechanics).

### Size Heuristic for Section C (Default)
- Compute total test count from REST descendants/assets traversal (preferred) or YAML traversal fallback.
- **Small** (<= 15 tests): include full suite/test hierarchy.
- **Medium** (16-50 tests): include all suites and a shortened test list (first 3 tests per suite) plus per-suite test counts.
- **Large** (> 50 tests): include suites only and per-suite test counts.
- User instruction always overrides heuristic.
- Heuristic bucket is internal and should not be printed in the default summary output.

### Setup Tests Placement Rule
- If `setUpTests` exists, list setup tests inside the root suite hierarchy.
- Setup tests are always shown first under root suite before regular tests/suites.

### Section D Labeling Rule
- In `Business Flow Summary`, label each flow by suite name directly (for example `Child Test Suite`) rather than `Flow (...)` prefixes.

## 8) Validation
- Cross-check that named suites/tools appear in both REST and YAML views.
- Ensure reported counts come from REST hierarchy traversal.
- Confirm notable values cited from `assets/data` also exist in YAML sections.
- If REST omits detail but YAML shows it, explicitly mark as "YAML-only evidence".
- Confirm active environment in summary matches environment configuration (`active: true`) in YAML/REST.
- If environment reference is used, confirm summary variable table values come from parsed `.env`/`.envs` content.

## 9) Failure Modes
- Name collisions (for example multiple `Untitled` data sources) obscure direct one-to-one mapping.
- Encoded path-like ids in REST can differ from display names.
- Partial summaries if only one source (REST or YAML) is used.
- Referenced environment file missing/unresolvable; active environment values cannot be expanded.

## 10) Safety / Rollback
- Read-only by default? Yes.
- No writes required.

## 11) Reuse Notes
- This card is the bridge from file operations to granular refactoring skills.
- Future L2/L3 cards should reference this card’s refactor target map format.
- Summaries should always distinguish structure-confidence (REST) from configuration-confidence (YAML).
- Default summary should prioritize user understanding of purpose, organization, and business flow over internal technical inventory.
