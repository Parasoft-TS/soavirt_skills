# Skill Status and Backlog
Purpose: provide a contributor-facing status ledger for current skill coverage plus a short active backlog.

Use this file to answer:
- what already exists,
- what is validated vs only defined,
- what is actively being improved next,
- what is deferred, retired, or still missing.

Do not use this file as the primary operator-facing routing surface.
- Use `docs/skills/skill-index.md` for routing and navigation.
- Use individual skill cards for endpoint-accurate behavior.

## Status Legend
- **Validated**: documented and backed by runtime evidence in this workspace/project history.
- **Defined**: card exists and is intended for use, but validation is partial, pending, or policy-level only.
- **In progress**: active hardening/validation work is underway.
- **Planned gap**: desired coverage does not yet have a stable card.
- **Deferred/Retired**: intentionally not being expanded right now.

## Active Priorities
- Harden and validate the new validation-enrichment orchestration path so happy-path bundle proposal, DB Tool resultset enrichment, multi-signal readiness heuristics, schema-source confirmation, and response-vs-database routing behave consistently:
  - `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
- Define and validate the new request-readiness remediation orchestration path so underconfigured existing/generated REST/SOAP client tests and DB Tools can be repaired before validation or later orchestration continues:
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
- Maintain and refine the API-first constrained REST Client same-operation edit path that uses REST Client `GET/PUT/GET` for constrained JSON request payloads and YAML fallback for same-operation resource/config edits, while keeping operation retargeting and migration work out of scope:
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
- Broaden Virtualize coverage beyond shared platform/policy surfaces.
- Convert remaining partially-defined validation/data-exchange cards into fully validated workflows:
  - `docs/skills/platform/skill-024-tst-create-from-raml.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`
- Decide whether the remaining structural-manipulation gaps should become standalone cards or be retired as unneeded complexity:
  - class-specific move/copy/rename/delete/reorder cards beyond the current family + existing validated coverage
- Continue refining contributor-facing canonical docs so `skill-index.md`, `backlog.md`, and workflow docs stay tightly aligned.
- Validate and tune the new cross-cutting XPath scalar-normalization policy in mixed XML/JSON tool chains:
  - `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`

## Skill Status Registry
### 1) Platform / File Operations
- `docs/skills/platform/skill-family-server-file-lifecycle.md`
  - **State:** Defined
  - **Notes:** canonical selection/anti-duplication surface for file-level lifecycle operations and shared rollback expectations.
- `docs/skills/platform/skill-001-shared-introspection.md`
  - **State:** Defined
  - **Notes:** canonical shared read/discovery surface; used broadly across runtime routing.
- `docs/skills/platform/skill-002-shared-file-transfer.md`
  - **State:** Validated
  - **Notes:** shared download/upload flow with no-BOM upload safety.
- `docs/skills/platform/skill-003-server-copy.md`
  - **State:** Validated
  - **Notes:** server-side file copy workflow with optional destination naming; foundation for rollback-safe server fallback copies.
- `docs/skills/platform/skill-004-server-rename.md`
  - **State:** Validated
  - **Notes:** in-place file rename workflow.
- `docs/skills/platform/skill-005-server-delete.md`
  - **State:** Validated
  - **Notes:** file delete workflow.
- `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - **State:** Defined
  - **Notes:** rollback-preserving local YAML edit composite intended for YAML fallback write branches such as constrained REST edits and structural fallback workflows.

### 2) Asset Creation / Generation
- `docs/skills/platform/skill-021-tst-create-empty.md`
  - **State:** Validated
  - **Notes:** empty `.tst` create/reuse path is stable.
- `docs/skills/platform/skill-022-tst-create-from-openapi.md`
  - **State:** Validated
  - **Notes:** OpenAPI/Swagger generation path is validated and hardened for ambiguous write recovery.
- `docs/skills/platform/skill-023-tst-create-from-wsdl.md`
  - **State:** Validated
  - **Notes:** WSDL generation path validated.
- `docs/skills/platform/skill-024-tst-create-from-raml.md`
  - **State:** Defined
  - **Notes:** endpoint contract validated; runtime validation still pending a reachable RAML source.
- `docs/skills/platform/skill-025-tst-create-from-xsd.md`
  - **State:** Validated
  - **Notes:** XSD generation path validated.

### 3) Read-Only Analysis
- `docs/skills/cross-cutting/skill-007-tst-content-summarization.md`
  - **State:** Defined
  - **Notes:** retained as the high-level human-friendly summary route.
- `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
  - **State:** Defined
  - **Notes:** foundational datasource discovery card; family-by-family validation still pending.
- `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - **State:** Defined
  - **Notes:** canonical deep YAML-based configuration/dataflow analysis route; representative runtime validation remains useful.

### 4) Structural Mutation
- `docs/skills/structure/skill-family-tst-object-manipulation.md`
  - **State:** Defined
  - **Notes:** architectural family/intent map, not a full endpoint card.
- `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
  - **State:** Defined
  - **Notes:** generalized move flow exists; YAML fallback branch now aligns to the shared rollback-preserving local-edit composite.
- `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
  - **State:** Defined
  - **Notes:** broad suite lifecycle card exists; targeted runtime validation can continue incrementally.
- `docs/skills/structure/skill-055-testsuite-create-from-wsdl.md`
  - **State:** Validated
  - **Notes:** suite-level WSDL generation workflow validated for disabled-environment generation, collision-aware managed generation, post-generation environment normalization into the original active environment, nested-parent placement readback, and a validated but non-default `referenceExistingEnvironment.file` branch that creates inactive reference nodes while runtime resolution still follows the active local environment.
- `docs/skills/structure/skill-047-generated-subset-prune.md`
  - **State:** Validated
  - **Notes:** generated-suite prune workflow is validated and reusable.

### 5) Client Tools
- `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
  - **State:** Validated
  - **Notes:** DB Tool lifecycle validated.
- `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - **State:** Validated
  - **Notes:** unconstrained REST Client creation/configuration is the current recommended path.
- `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`
  - **State:** Defined
  - **Notes:** first-pass SOAP Client lifecycle card covering the API-exposed HTTP request/transport/misc surface; validated on scaffold/readback, WSDL-originated-compatible copy/update preservation, and `Response SOAP Envelope` output mapping, while full WSDL-tab parity and non-HTTP transport authoring remain pending.
- `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - **State:** Validated
  - **Notes:** API-first same-operation edit path for existing constrained REST Clients; validated in contributor research for same-operation path/query value edits, semantically equivalent base/service-definition literalization, and constrained `Form JSON` request payload editing through REST Client `GET/PUT/GET`, with the YAML fallback branch intended to reuse the shared rollback-preserving local-edit composite.

### 6) Validation Tools
- `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - **State:** Defined
  - **Notes:** create path validated; modify/delete/copy coverage still partial.
- `docs/skills/validation/skill-016-xml-assertor-workflow.md`
  - **State:** Validated
  - **Notes:** XML Assertor workflow validated.
- `docs/skills/validation/skill-029-json-validator-workflow.md`
  - **State:** Defined
  - **Notes:** card exists; runtime validation remains pending.
- `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - **State:** Defined
  - **Notes:** card exists; runtime validation remains pending.
- `docs/skills/validation/skill-031-diff-tool-workflow.md`
  - **State:** Defined
  - **Notes:** card exists; per-mode ignore-setting/runtime matrix validation remains pending.

### 7) Data Exchange Tools
- `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`
  - **State:** Validated
  - **Notes:** XML Data Bank extraction validated.
- `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - **State:** Defined
  - **Notes:** JSON Data Bank card exists; runtime validation remains pending.

### 8) Execution Diagnostics
- `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - **State:** Validated
  - **Notes:** canonical execution payload/results/traffic baseline.
- `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - **State:** Validated
  - **Notes:** focused failure diagnosis from runtime traffic is validated.

### 9) Cross-Cutting Policy
- `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - **State:** Defined
  - **Notes:** policy card for XPath-over-JSON selector semantics.
- `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - **State:** Defined
  - **Notes:** deterministic naming policy is established.
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - **State:** Validated
  - **Notes:** core chaining semantics are established and referenced broadly.
- `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - **State:** Validated
  - **Notes:** canonical parent/output mapping authority.
- `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - **State:** Defined
  - **Notes:** policy-level card for tool-managed request headers.
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - **State:** Validated
  - **Notes:** read-merge-write policy promoted from observed tool PUT behavior.
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - **State:** Defined
  - **Notes:** canonical runtime API preflight policy; broadly active as a workflow dependency.
- `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`
  - **State:** Defined
  - **Notes:** policy-level counterpart to Skill 049 for non-tool objects.
- `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`
  - **State:** Defined
  - **Notes:** centralizes XML scalar-selector `/text()` normalization guidance with explicit JSON boundary behavior.

### 10) Composite Orchestration
- `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`
  - **State:** In progress
  - **Notes:** canonical underspecified-intent entry point for service-test authoring; now focuses on ambiguity resolution, generation-critical pre-write gates, and high-level phase sequencing, while routing repeated clarified sub-workflows to Skills 056, 057, and 058 instead of continuing to absorb detailed branch policy.
- `docs/skills/composite-orchestration/skill-056-single-client-authoring-intent-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for one-client authoring intent across endpoint-only and contract-informed REST/SOAP flows; routes to Skills 020 and 034 rather than broad generation cards.
- `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for adding validation to existing or newly generated tests and DB Tool resultsets, with live-runtime-evidence gating, multi-signal readiness fail-closed behavior, an explicit remediation-approval-vs-validation-approval boundary, happy-path schema-plus-content bundle defaults for API responses, DB Tool assertion/diff enrichment defaults, conservative negative-test policy, and response-vs-database orchestration.
- `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for detecting and repairing underconfigured existing/generated REST/SOAP client tests and DB Tools before validation or later orchestration proceeds; now enforces a strict candidate-value sourcing ladder (user/session/contract/same-`.tst` only by default), best-guess proposal gating, and no autonomous live-service probing unless the user explicitly approves it, while supporting REST-only, SOAP-only, DB-only, and mixed remediation slices and keeping SQL strategy and DB-side setup out of scope.

## Coverage Gaps / Planned New Cards
- Structural manipulation coverage beyond current validated cards remains incomplete if the project still wants fine-grained standalone cards for:
  - class-specific move flows
  - class-specific copy flows
  - class-specific rename flows
  - class-specific delete flows
  - explicit reorder cards/overlays
- Virtualize-specific skill coverage remains shallow beyond shared platform/policy surfaces.
- Additional datasource-family validation evidence is still desirable for Skill 051 and downstream datasource-aware workflows.
- Repeated-execution response-volatility profiling for validation-bundle selection remains future work rather than part of the v1 validation-enrichment orchestration.
- Higher-scope constrained REST Client work remains future research, including operation retargeting within the same service definition and longer-term v1-to-v2 OpenAPI migration orchestration with downstream chained-tool adaptation.

## Deferred / Retired
- REST Client constrained creation cards (`Skills 026/027`)
  - **State:** Deferred/Retired
  - **Notes:** current recommended path remains unconstrained REST Client creation via `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.

## Virtualize Future Track
- Deeper `.pva` lifecycle and deployment-aware workflows.
- Virtualize-specific validation of shared policies already written at the cross-cutting/platform layers.
- Additional operator-facing routing once Virtualize coverage grows beyond the current shared foundation.
