# Skill Index

Purpose: quickly locate only the skills needed for a task to control context size.

## Layer L0: Platform / File Operations
- `platform/skill-001-shared-introspection.md`
- `platform/skill-002-shared-file-transfer.md`
- `platform/skill-003-server-copy-rename.md`
- `platform/skill-004-server-rename.md`
- `platform/skill-005-server-delete.md`

## Layer L0C: Composite File Workflows
- `platform/skill-006-safe-file-refactor-composite.md`

## Layer L0O: Composite Orchestration Workflows
- `composite-orchestration/skill-033-service-test-intent-orchestration.md` (intent intake, scope discovery, and cross-skill execution planning for service test authoring)

## Layer L1: Asset Operations (Planned)
- `cross-cutting/skill-007-tst-content-summarization.md`
- `platform/skill-021-tst-create-empty.md` (create brand-new empty `.tst` via `POST /v6/files/tsts`)
- `platform/skill-022-tst-create-from-openapi.md` (create `.tst` from OpenAPI/Swagger via `POST /v6/files/tsts/swagger`)
- `platform/skill-023-tst-create-from-wsdl.md` (create `.tst` from WSDL via `POST /v6/files/tsts/wsdl`)
- `platform/skill-024-tst-create-from-raml.md` (create `.tst` from RAML via `POST /v6/files/tsts/raml`)
- `platform/skill-025-tst-create-from-xsd.md` (create `.tst` from XSD via `POST /v6/files/tsts/xsd`)
- `cross-cutting/skill-011-xpath-over-json-query-semantics.md` (cross-cutting selector rule for JSON tools)
- `cross-cutting/skill-013-test-naming-policy.md` (cross-cutting deterministic unique naming for execution targeting)
- `cross-cutting/skill-017-output-chaining-model.md` (cross-cutting output-provider semantics for tool chaining)
- `cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (living lookup table for tool outputs and default chaining anchors)
- `cross-cutting/skill-032-client-header-ownership.md` (cross-cutting request-header ownership policy for client tools)
- `cross-cutting/skill-049-tool-put-read-merge-write-policy.md` (cross-cutting safe update policy for existing tool PUT operations)
- `cross-cutting/skill-050-server-api-capability-preflight.md` (cross-cutting endpoint-method compatibility preflight and fallback routing)
- Parse `.pva` top-level structure

## Layer L2: Structural Refactors (Planned)
- `structure/skill-family-tst-object-manipulation.md` (umbrella routing + dependency map)
- `structure/skill-008-datasource-type-targeted-move.md` (generalized move for any datasource implementation type)
- `structure/skill-009-testsuite-creation-and-configuration.md` (staged suite create/update option coverage)
- `structure/skill-047-generated-subset-prune.md` (prune non-selected operation suites after service-definition generation)
- `validation/skill-010-json-assertor-workflow.md` (JSON Assertor create/modify/delete/copy)
- `execution-diagnostics/skill-012-test-execution-xml-report.md` (queue test run + poll + retrieve XML runtime report; triad step 1-2 baseline)
- `execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md` (focused failure triage using results XML + runtime traffic; triad step 3 + correlation)
- `client-tools/skill-015-db-tool-lifecycle.md` (DB Tool create/update/copy/move/delete with deterministic suite placement)
- `validation/skill-016-xml-assertor-workflow.md` (cross-tool XML Assertor lifecycle: create/read/update/delete/copy + runtime validation)
- `validation/skill-029-json-validator-workflow.md` (cross-tool JSON Validator create/configure/run-validation on runtime JSON output)
- `validation/skill-030-xml-validator-workflow.md` (cross-tool XML Validator create/configure/run-validation on runtime XML output)
- `validation/skill-031-diff-tool-workflow.md` (Diff Tool create/configure with XML/JSON/Text/Binary mode-specific ignore settings)
- `data-exchange/skill-019-xml-databank-extraction-workflow.md` (cross-tool XML Data Bank lifecycle: create/read/update/delete/copy + runtime extraction validation)
- `data-exchange/skill-028-json-databank-extraction-workflow.md` (cross-tool JSON Data Bank lifecycle: create/read/update/delete/copy + runtime extraction validation)
- `client-tools/skill-020-rest-client-none-mode-workflow.md` (REST Client lifecycle in None mode: create/read/update/delete/copy + execution validation)
- Atomic operation cards (planned): copy, move, reorder-children, rename, delete
- Object-class overlays (planned): suites, tools, data sources, environments, global properties
- Rename test suite/test nodes
- Move/reorder suite children

## Layer L3: Configuration Refactors (Planned)
- Edit specific tool configuration fields
- Update test/suite execution settings

## Layer L4: Semantic / Policy Refactors (Planned)
- Convention-driven naming updates
- Cross-asset consistency rules

## Dependency Guidance
- L1 depends on L0.
- L2 depends on L1 + selected L0 write operations.
- L3 depends on L1 + selected L0 write operations.
- L4 depends on L2/L3 and should not bypass validation steps.

## Selection Heuristic
1. Identify target outcome and asset type.
2. Start with the highest relevant layer card.
3. Load only its declared dependencies.
4. Execute and log evidence; avoid pulling unrelated cards.

## Intent-Domain View (Alternative Navigation)

Use this view when the user asks by capability (for example "client tool" or "validation") rather than by endpoint/layer.

### Platform & Asset Lifecycle
- `platform/skill-001-shared-introspection.md`
- `platform/skill-002-shared-file-transfer.md`
- `platform/skill-003-server-copy-rename.md`
- `platform/skill-004-server-rename.md`
- `platform/skill-005-server-delete.md`
- `platform/skill-006-safe-file-refactor-composite.md`
- `platform/skill-021-tst-create-empty.md`
- `platform/skill-022-tst-create-from-openapi.md`
- `platform/skill-023-tst-create-from-wsdl.md`
- `platform/skill-024-tst-create-from-raml.md`
- `platform/skill-025-tst-create-from-xsd.md`

### Composite Orchestration
- `composite-orchestration/skill-033-service-test-intent-orchestration.md`

### Structural Authoring
- `structure/skill-family-tst-object-manipulation.md`
- `structure/skill-008-datasource-type-targeted-move.md`
- `structure/skill-009-testsuite-creation-and-configuration.md`
- `structure/skill-047-generated-subset-prune.md`

### Client Tools
- `client-tools/skill-020-rest-client-none-mode-workflow.md`
- `client-tools/skill-015-db-tool-lifecycle.md`

### Data Exchange Tools
- `data-exchange/skill-019-xml-databank-extraction-workflow.md`
- `data-exchange/skill-028-json-databank-extraction-workflow.md`

### Validation Tools
- `validation/skill-010-json-assertor-workflow.md`
- `validation/skill-016-xml-assertor-workflow.md`
- `validation/skill-029-json-validator-workflow.md`
- `validation/skill-030-xml-validator-workflow.md`
- `validation/skill-031-diff-tool-workflow.md`

### Execution & Diagnostics
- `execution-diagnostics/skill-012-test-execution-xml-report.md`
- `execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`

### Cross-Cutting Semantics & Policy
- `cross-cutting/skill-007-tst-content-summarization.md`
- `cross-cutting/skill-011-xpath-over-json-query-semantics.md`
- `cross-cutting/skill-013-test-naming-policy.md`
- `cross-cutting/skill-017-output-chaining-model.md`
- `cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
- `cross-cutting/skill-032-client-header-ownership.md`
- `cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- `cross-cutting/skill-050-server-api-capability-preflight.md`
