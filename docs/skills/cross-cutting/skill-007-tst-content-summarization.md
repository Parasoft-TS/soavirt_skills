# Skill Card 007: TST Content Summarization and Intent Mapping
## 1) Skill Name
TST content summarization (local-path authoritative, YAML-authoritative)
## 2) Objective
- Produce an accurate, user-friendly summary of what a `.tst` file contains and what it is doing.
- Build a translation layer between user intent language and concrete editable targets in the `.tst`.
- Highlight the application/service interface under test and the currently active environment.
- Use the local `.tst` YAML as the source of truth for structure and configuration details.
## 3) Layer
- L1 read-only analysis
## 4) Dependencies
- Required when the target `.tst` path is unresolved:
  - `docs/skills/platform/skill-001-shared-introspection.md`
- Additive:
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md` when active project context is available
## 5) Inputs
- Required:
  - explicit local `.tst` path, or
  - enough file-backed identity to resolve one local `.tst` through Skill 001
- Optional:
  - `audience` (`technical`, `mixed`, `non-technical`)
  - `focusAreas` (for example `environments`, `data sources`, `assertions`, `chained tests`)
  - active project context
## 6) Procedure
0. Resolve the local `.tst` target:
   - if an explicit local path is provided, use it directly
   - otherwise resolve one local `.tst` through Skill 001 using active-project-first and likely-root-first search
   - if multiple plausible local targets remain, fail closed and ask the user to clarify
1. Read the local `.tst` YAML.
2. Parse YAML structure and configuration:
   - suite/test/tool hierarchy
   - names, `testID`, and nested tool chains
   - notable configuration details such as endpoints, parameterization, validations, and data sources
3. If YAML indicates `EnvironmentReference`, resolve the referenced `.env` or `.envs` locally:
   - use the referenced relative/local path when present
   - otherwise search active project locations first through Skill 001-style local resolution
   - parse the referenced file and capture its environment entries as local reference evidence only
   - do not treat the presence of a referenced environment file as proof that runtime has switched to it
   - enrich Section B with referenced-file context and local file values only when they are clearly labeled as reference-file content rather than verified active-environment values
4. Produce three outputs:
   - Structural summary (what exists)
   - Behavioral summary (what it appears to do)
   - Refactor target map (where common requested edits would apply)
5. Fail-closed enforcement:
   - if summary claims rely on anything other than local YAML or referenced local environment-file content, restart the analysis using only allowed evidence
## 6.1) Local YAML-Evidence Boundary Policy (Required)
- Primary analysis evidence for this skill must come from the local `.tst` YAML content.
- Before local file read begins, only local path-resolution steps are allowed when needed to resolve the file target.
- After local file read begins, do not use API discovery or runtime introspection as summary evidence.
- Environment-reference exception:
  - resolving referenced `.env` / `.envs` is allowed for Section B via local file resolution and local file reads only.
  - variable table values in Section B may be reported from local file content only when they are explicitly labeled as referenced-file content rather than verified active-environment state.
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
  - render only sections `A`, `B`, `C`, `D` in this exact order
  - do not replace, rename, or omit template sections in default mode
  - treat the template's output-profile scaffold as internal planning guidance only; do not render it in the default user-facing summary
- Response validity gate:
  - output is invalid unless sections `A`, `B`, `C`, `D` appear in exact template order
  - unresolved required fields must be emitted as `Unknown (<reason>)`
- Include protocol/transport deep detail only when it helps answer the user's question.
- Exclude risk/opinion/recommendation sections by default unless the user explicitly asks.
## 7.1) Required Fields by Section (Default)
- Internal output-profile planning may set depth/voice for the response, but it must not be emitted as a separate visible section in default mode.
- Section `A) Executive Summary` must include:
  - what the `.tst` does (1-3 sentences)
  - application/service interface under test
  - active environment used for testing
  - primary business flow(s) covered
  - flow model
- Section `B) Tested Environment (Active)` must include:
  - active environment name
  - variable/value table header and rows
  - when reference mode is active, include source reference file context (`.env` or `.envs`) and clearly label any referenced-file values that are shown as reference evidence rather than proof of the active runtime environment
- Section `C) Structure at a Glance` must include:
  - local file path
  - root suite name
  - active environment
  - suite/test hierarchy output according to size heuristic
- Section `D) Business Flow Summary` must include:
  - one bullet group per represented flow suite
  - flow labels must be suite names
## 7.2) Missing-Data Fallback Policy (Required)
- Never omit a required section because data is missing.
- If a required field cannot be resolved, keep the field and set value to `Unknown`.
- Add a short reason inline.
## Size Heuristic for Section C (Default)
- Compute total test count from YAML traversal.
- **Small** (<= 15 tests): include full suite/test hierarchy.
- **Medium** (16-50 tests): include all suites and a shortened test list (first 3 tests per suite) plus per-suite test counts.
- **Large** (> 50 tests): include suites only and per-suite test counts.
- User instruction always overrides heuristic.
## 8) Validation
- Ensure reported suites/tests/tools are present in local YAML hierarchy.
- Ensure reported counts come from YAML traversal.
- Confirm notable values in the summary exist in local YAML fields.
- Confirm active environment in summary matches environment configuration evidenced in local YAML; when only referenced environment-file content is available, report it as reference evidence rather than as proof of active runtime selection.
- Mandatory pre-response compliance checklist:
  1. local `.tst` target resolved
  2. local `.tst` YAML read successfully
  3. no API/runtime introspection was used as analysis evidence after local file read began
  4. template sections `A/B/C/D` and required fields are present in exact order
  5. reported structure/flows/values include local YAML evidence anchors
  6. if `EnvironmentReference` is used, Section B distinguishes local `.env` / `.envs` reference evidence from verified active-environment evidence
## 9) Failure Modes
- Multiple plausible local `.tst` targets remain and the card guesses instead of clarifying.
- Referenced environment file is missing or not resolvable locally.
- Very large YAML files require scoped/depth-limited summaries to remain readable.
## 10) Safety / Rollback
- Read-only by default? Yes
- No writes required.
## 11) Reuse Notes
- This card is the bridge from local file-backed analysis to granular refactoring skills.
- Future mutation cards should reuse its refactor target map concepts, but not its summary template.
- Default summaries should prioritize user understanding of purpose, organization, and business flow over internal technical inventory.
