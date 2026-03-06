# Skill 017: SOAtest Output Chaining Model (Cross-Cutting)

## 1) Skill Name
SOAtest tool output chaining model and parent-anchor selection

## 2) Objective
Provide deterministic rules for selecting the correct output-provider parent when chaining tools, so assertions/validators/databanks and other chainable tools bind to semantic outputs (for example response/results) rather than diagnostic outputs (for example traffic viewer channels).

## 3) Core Model
- A tool (for example REST Client, DB Tool, XML Assertor) can be:
  - standalone under a suite, or
  - chained to another tool’s output.
- Tools expose multiple output channels (for example request, response, results, traffic object).
- Chaining composes behavior by binding child tools to a specific output channel.

## 4) Output Semantics Rule
- Prefer semantic data outputs for chaining logic:
  - REST workflows: response-oriented outputs for JSON/XML assertors/validators/databanks and other JSON tools.
  - DB workflows: `Results as XML` for XML assertors, XML databanks, and other XML tools.
- Apply strict media-type gate before tool-family selection:
  - JSON tools only when runtime response is JSON,
  - XML tools only when runtime response is XML,
  - plain-text responses should route to text-oriented comparison (for example Diff Tool text mode) unless user explicitly requests another strategy.
- Treat `Traffic Object` as a diagnostics output by default:
  - primary consumer: `Traffic Viewer`
  - rare exception: specialized tooling (for example write file tool)
  - do not use for assertor, diff, validator, or data bank chaining unless explicitly required.

## 5) Parent Selection Procedure
1. Identify candidate output-provider children under the producer tool.
  - enumerate from the producer tool/output graph directly; do not infer from sibling tools.
2. Select by output semantics first (response/results), not by name familiarity alone.
3. Create child tool under that output-provider parent.
4. Run focused execution and verify that assertion input matches expected payload type.
5. If parser/type mismatch appears (for example XML parser on plain text), re-check output parent before changing assertion logic.
6. If payload type is plain text and validation intent remains, switch to Diff Tool text mode seeded from baseline response content.

### 5.1) Anti-Inference Guard (Required)
- Never use sibling presence (for example `Traffic Viewer`) as evidence of correct chaining parent.
- `Traffic Viewer` visibility is not a semantic output selector.
- If semantic response/results outputs are not resolvable from producer context, stop and gather additional output-discovery evidence before attaching tools.

### 5.2) Output Provider Non-Discoverability (Required)
- Output providers that have no child tools are NOT returned by `/children` or `/descendants/assets`.
- Do not attempt to discover output provider IDs by enumerating children of the producer tool.
- Instead, construct the output provider path directly using known patterns:
  - REST Client: `<rest-client-id>/Response Traffic`
  - DB Tool: `<db-tool-id>/Results as XML`
- See `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` Section 5 for the canonical path construction reference.

### 5.3) Canonical Map + Fail-Closed Rule (Required)
- Treat `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` as the single source of truth for producer/output-provider mapping.
- Do not hardcode new producer-to-parent mappings in downstream workflow cards.
- If the producer/output pair is missing from Skill 018, stop and request a Skill 018 update before chaining.
- Do not guess or infer an unregistered output-provider parent path.

## 6) Validation Signals
- Correct binding indicators:
  - assertion or databank tool receives payload type matching its parser (JSON/XML/etc.)
  - business assertion pass/fail reflects content expectation, not parser bootstrap errors.
- Misbinding indicators:
  - parser/prolog/syntax errors despite valid XPath/query
  - assertion appears to evaluate request text/metadata instead of semantic response/results payload.

## 7) Failure Modes
- Chaining to `Traffic Object` by default causes parser/type mismatches for business assertions.
- Assuming all visible children are equivalent output anchors.
- Fixing selector syntax before validating output binding.
- Choosing chain parent from sibling tool artifacts instead of producer output discovery.

## 8) Safety / Rollback
- Write impact: creates/moves child tools in suite structure.
- Rollback:
  - move tool to correct output parent (`POST /v6/tools/move`), or
  - delete and recreate under correct parent.

## 9) Reuse Notes
- Applies to SOAtest: Yes (validated pattern).
- Applies to Virtualize: conceptually relevant; endpoint-level validation pending.
- Load this card as a dependency before building any chained tool skill.
- Companion reference:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - use as the first lookup for concrete output-channel defaults.
- Maintenance contract:
  - when output-chaining policy changes, update this card and Skill 018 in the same session.

## 10) Validation Snapshot (2026-03-03)
- DB Tool case:
  - incorrect chain: `.../Traffic Object/XML Assertor - DB Row ID` produced XML parser/prolog error.
  - corrected chain: `.../Results as XML/XML Assertor - DB Row ID` removed parser error and passed DB test assertion.