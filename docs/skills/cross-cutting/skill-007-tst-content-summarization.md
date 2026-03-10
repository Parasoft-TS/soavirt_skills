# Skill Card 007: TST Content Summarization and Intent Mapping

## 1) Skill Name
TST Content Summarization (YAML-Authoritative)

## 2) Objective
- Produce an accurate, user-friendly summary of what a `.tst` file contains and what it is doing.
- Build a translation layer between user intent language and concrete editable targets in the TST.
- Highlight the application/service interface under test and the currently active environment.
- Use downloaded YAML as the source of truth for structure and configuration details.

## 3) Layer
- L1 Asset Operations

## 4) Dependencies
- `skill-002-shared-file-transfer.md` (download current YAML safely)

## 5) Inputs
- Required:
  - `tstFileId`
- Optional:
  - `audience` (`technical`, `mixed`, `non-technical`)
  - `focusAreas` (for example `environments`, `data sources`, `assertions`, `chained tests`)

## 6) Procedure
0. Phase A - target resolution (if target identity is unresolved):
   - allowed endpoints: `GET /v6/children`, `GET /v6/descendants/files`
   - use only to resolve `tstFileId` and optional focus context.
   - once `tstFileId` is resolved, transition to Phase B.
1. Phase B - download YAML via `GET /v6/files/download?id=<tstFileId>`.
2. Parse YAML structure and configuration:
   - suite/test/tool hierarchy
   - names, `testID`, and nested tool chains
   - notable configuration details (endpoints, parameterization, validations, data sources)
3. If YAML indicates `EnvironmentReference`, resolve external dependency:
  - find referenced `.env` or `.envs` file on server
  - download dependency file
  - parse XML and resolve active environment
    - `.env`: single environment node
    - `.envs`: multiple environment nodes; select by `activeEnvironmentName`
  - enrich Section B with active environment variables from external file
4. Produce three outputs:
   - Structural summary (what exists)
   - Behavioral summary (what it appears to do)
   - Refactor target map (where common requested edits would apply)
5. Fail-closed enforcement:
   - if any forbidden post-download endpoint is called (see 6.1), abort and restart Phase B using YAML evidence only.

## 6.1) YAML-Evidence Boundary Policy (Required)
- Primary analysis evidence for this skill must come from downloaded YAML content.
- Before YAML download: only target-resolution reads are allowed (`/v6/children`, `/v6/descendants/files`) when needed.
- After YAML download begins: do not use API discovery/introspection calls as analysis evidence.
- Forbidden post-download endpoints (non-exhaustive):
  - `/v6/children`
  - `/v6/descendants/files`
  - `/v6/descendants/assets`
  - `/v6/assets/data`
  - `/v6/tools/*`
  - `/v6/datasources/*`
- Environment reference exception:
  - resolving referenced `.env`/`.envs` is allowed for Section B via filesystem-level resolution and `GET /v6/files/download` only.
  - variable table values in Section B must still be reported from downloaded file content.

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
- Template compliance is mandatory for default summaries:
  - output sections must appear in this exact order: `0`, `A`, `B`, `C`, `D`
  - do not replace, rename, or omit template sections in default mode
- Response validity gate:
  - output is invalid unless sections `0`, `A`, `B`, `C`, `D` appear in exact template order.
  - freeform/non-template summaries are not allowed for default Skill 007 output.
  - required section fields must be present; unresolved fields must be emitted as `Unknown (<reason>)`.
- Include protocol/transport deep detail only when it helps answer the user’s question.
- Exclude risk/opinion/recommendation sections by default unless user explicitly asks.
- In Section B, include source context (for example referenced `.envs` filename) but omit implementation-detail narration (for example "downloaded/parsing" mechanics).

### 7.1) Required Fields by Section (Default)
- Section `0) Output Profile` must include:
  - `Audience`, `Depth`, `Voice`
- Section `A) Executive Summary` must include:
  - what the TST does (1-3 sentences)
  - application/service interface under test
  - active environment used for testing
  - primary business flow(s) covered
  - flow model
- Section `B) Tested Environment (Active)` must include:
  - active environment name
  - variable/value table header and rows
  - when reference mode is active, include source reference file context (`.env` or `.envs`)
- Section `C) Structure at a Glance` must include:
  - file id
  - root suite name
  - active environment
  - suite/test hierarchy output according to size heuristic
- Section `D) Business Flow Summary` must include:
  - one bullet group per represented flow suite
  - flow labels must be suite names

### 7.2) Missing-Data Fallback Policy (Required)
- Never omit a required section because data is missing.
- If a required field cannot be resolved, keep the field and set value to `Unknown`.
- Add a short reason inline (for example: `Unknown (referenced environment file not resolvable)`).
- Apply this policy per-field; do not downgrade the full summary to freeform prose.

### Size Heuristic for Section C (Default)
- Compute total test count from YAML traversal.
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

### 7.3) Common Divergence Traps (Required Handling)
- Trap: continuing API asset introspection after target resolution.
  - Required handling: stop API exploration and continue with YAML evidence.
- Trap: returning narrative summary outside the template.
  - Required handling: regenerate output in the required `0/A/B/C/D` template structure.
- Trap: mixing discovery evidence and summary evidence without source distinction.
  - Required handling: use API reads only for target resolution; all summary claims must be YAML-evidenced (plus `.env`/`.envs` file content for Section B when referenced).

## 8) Validation
- Ensure reported suites/tests/tools are present in YAML hierarchy.
- Ensure reported counts come from YAML traversal.
- Confirm notable values in the summary (for example URLs, variables, selectors) exist in YAML fields.
- Confirm active environment in summary matches environment configuration in YAML.
- If environment reference is used, confirm summary variable table values come from parsed `.env`/`.envs` content.
- Mandatory pre-response compliance checklist:
  1. `tstFileId` resolved.
  2. `.tst` YAML downloaded via `/v6/files/download`.
  3. No forbidden post-download API calls were used as analysis evidence.
  4. Template sections `0/A/B/C/D` and required fields are present in exact order.
  5. Reported structure/flows/values include YAML evidence anchors.
  6. If `EnvironmentReference` is used, Section B values are evidenced from downloaded `.env`/`.envs` content.

## 9) Failure Modes
- Name collisions (for example multiple `Untitled` data sources) can reduce summary specificity.
- Parallel naming fields in YAML (for example test node name vs tool display name) may diverge.
- Very large YAML files can require scoped/depth-limited summaries to remain readable.
- Referenced environment file missing/unresolvable; active environment values cannot be expanded.

## 10) Safety / Rollback
- Read-only by default? Yes.
- No writes required.

## 11) Reuse Notes
- This card is the bridge from file operations to granular refactoring skills.
- Future L2/L3 cards should reference this card’s refactor target map format.
- Summaries are YAML-authoritative for both structure and configuration.
- Default summary should prioritize user understanding of purpose, organization, and business flow over internal technical inventory.
