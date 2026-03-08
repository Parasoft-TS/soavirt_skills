# Skill Index

Purpose: provide a clean operator-facing entry point for selecting the minimum required skills for a task.

This file is the primary routing surface for skill selection.
- Use this file to choose the right skill family and target card for the current task.
- Use `docs/skills/backlog.md` for status, validation state, and planned work.
- Use `docs/workflow/agent-workflow.md` for global loading, safety, and failure-handling policy.

## Quick Intent Cues (Parasoft-Domain Trigger)
Use skill routing cue policy from `docs/workflow/agent-workflow.md` under `Parasoft Intent Detection Gate (Global)`.

If cues are absent and the request appears general-purpose/non-Parasoft, do not force skill routing; ask one clarification question when uncertain.

## Practical Routing Registry

### A) Service-Test Authoring
Route to `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` when the user wants to:
- create new service tests
- generate tests from OpenAPI/WSDL/RAML/XSD
- turn an endpoint/service definition into a `.tst`
- gather scope and plan before test authoring

### B) Read-Only TST Analysis
Use this registry before selecting overlapping read-only analysis cards for `.tst` requests.

#### Route to `docs/skills/cross-cutting/skill-007-tst-content-summarization.md`
Use when the user asks:
- what a `.tst` does
- what flows it covers
- what service/interface it tests
- for a human-friendly or high-level explanation

Preferred prompts:
- `summarize this tst`
- `what does this test suite do?`
- `give me a high-level explanation`

#### Route to `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
Use when the user asks:
- what data sources exist
- what type a data source is
- what columns are available
- what sample rows are available
- for datasource-focused introspection without full suite configuration tracing

Preferred prompts:
- `what data sources are in this suite?`
- `what columns can I parameterize with?`
- `show me datasource details`

#### Route to `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
Use when the user asks:
- how a suite/test/tool is configured
- where a dynamic value comes from
- where a variable is used
- what validations are applied
- for execution context, step flow, or dataflow analysis

Preferred prompts:
- `analyze this suite`
- `how is this configured?`
- `trace the dataflow`
- `where does this variable come from?`
- `what validations/assertors are applied?`

#### Tie-Break Rule (Required)
- If an analysis prompt could match both Skill 007 and Skill 052, prefer Skill 052 unless the user explicitly asks for a high-level or human-friendly summary.
- If a prompt is specifically about datasource inventory, columns, or sample rows, prefer Skill 051 unless the user also asks for broader suite configuration/dataflow analysis.
- Once Skill 052 is selected, its YAML-only evidence policy and template contract are binding.

## Primary Navigation by Skill Family

### 1) Platform / File Operations
Use for workspace-level discovery and file lifecycle operations.

- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/platform/skill-002-shared-file-transfer.md`
- `docs/skills/platform/skill-003-server-copy-rename.md`
- `docs/skills/platform/skill-004-server-rename.md`
- `docs/skills/platform/skill-005-server-delete.md`
- `docs/skills/platform/skill-006-safe-file-refactor-composite.md`

### 2) Asset Creation / Generation
Use for creating new `.tst` assets from scratch or from service definitions.

- `docs/skills/platform/skill-021-tst-create-empty.md`
- `docs/skills/platform/skill-022-tst-create-from-openapi.md`
- `docs/skills/platform/skill-023-tst-create-from-wsdl.md`
- `docs/skills/platform/skill-024-tst-create-from-raml.md`
- `docs/skills/platform/skill-025-tst-create-from-xsd.md`

### 3) Read-Only Analysis
Use for explanation, configuration tracing, and datasource discovery without mutation.

- `docs/skills/cross-cutting/skill-007-tst-content-summarization.md`
- `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
- `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`

### 4) Structural Mutation
Use for suite/object creation, movement, reordering, and subset pruning.

- `docs/skills/structure/skill-family-tst-object-manipulation.md`
- `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
- `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
- `docs/skills/structure/skill-047-generated-subset-prune.md`

### 5) Client Tools
Use for client-tool lifecycle and configuration.

- `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
- `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`

### 6) Validation Tools
Use for validators, assertors, and diff-based comparison.

- `docs/skills/validation/skill-010-json-assertor-workflow.md`
- `docs/skills/validation/skill-016-xml-assertor-workflow.md`
- `docs/skills/validation/skill-029-json-validator-workflow.md`
- `docs/skills/validation/skill-030-xml-validator-workflow.md`
- `docs/skills/validation/skill-031-diff-tool-workflow.md`

### 7) Data Exchange Tools
Use for databank extraction and data propagation.

- `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`
- `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`

### 8) Execution Diagnostics
Use for running tests, retrieving results, and diagnosing failures.

- `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`

### 9) Cross-Cutting Policy / Semantics
Use these as required dependencies when a target skill calls for them. These are usually not first-choice entry points unless the user asks directly about the policy topic.

#### Naming / Query / Semantics
- `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
- `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
- `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`

#### Output Chaining / Tool Attachment
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
- `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`

#### Client / Mutation Safety
- `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`

#### Capability / Compatibility
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`

### 10) Composite Orchestration
Use for multi-step conversational intake and cross-skill planning/execution.

- `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`

## Selection Heuristic
1. For each new user prompt, re-run this routing heuristic before reusing any previously selected target card.
2. Determine whether the request is:
   - authoring/generation
   - read-only analysis
   - structural mutation
   - tool lifecycle/configuration
   - execution diagnostics
3. Apply any explicit routing registry above.
4. If no explicit routing rule applies, choose the smallest matching skill family from the Primary Navigation section.
5. Load the target card and its declared dependencies.
6. For server-API-mediated Parasoft requests, also apply the workflow-mandated runtime prelude from `docs/workflow/agent-workflow.md`; for write-capable requests, also apply the operation-class bundles.
7. Avoid loading unrelated cards.

## Architectural Layering Model (Secondary View)
This section is for dependency reasoning and long-term architecture. It is not the primary operator-facing navigation model.

- **L0 Platform**
  - workspace/file primitives such as list, download, upload, copy, rename, delete
- **L1 Read / Inspect**
  - summarization, datasource discovery, configuration analysis
- **L2 Structural Mutation**
  - create/move/reorder/prune suites, tools, and datasources
- **L3 Configuration Mutation**
  - configure tool/test behavior and settings
- **L4 Semantic / Orchestration**
  - cross-skill orchestration, policy-driven flows, convention-level behavior

### Layer Notes
- Not every skill family maps cleanly to a single layer.
- `Execution Diagnostics` and `Cross-Cutting Policy` are shared families that support multiple layers.
- Use `docs/skills/backlog.md` for planned future expansion of the layer model rather than adding planned placeholders here.
