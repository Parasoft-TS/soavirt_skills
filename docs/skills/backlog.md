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
- Define and thread the canonical SOAtest environment owner so internal environments, referenced environment nodes, external project `.env` / `.envs` files, and active-environment verification all have one reusable runtime owner:
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
- Harden the project-aware runtime contract so direct atomic routing still inherits active project context for placement, source/value resolution, validation priorities, and environment-sensitive work:
  - `docs/workflow/agent-workflow.md`
  - `AGENTS.md`
  - `docs/skills/skill-index.md`
- Harden and validate Skill 033 so template-backed approval/completion artifacts, autonomous defaults, dynamic standard-negative planning, separate interactive security intake, and always-on validation sequencing behave consistently:
  - `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`
- Implement and harden the additive explicit opt-in experimental exploration-first broad-authoring lane so direct API exploration, exploration-backed validation, broad operation-level security targeting, and bounded post-authoring correction have clear owners without weakening stable `033/057/058`:
  - `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md`
  - `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
  - `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`
- Harden and validate the validation-enrichment orchestration path so happy-path bundle proposal, DB Tool resultset enrichment, multi-signal readiness heuristics, schema-source confirmation, negative/security exception handling, and response-vs-database routing behave consistently:
  - `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
- Define and validate the Penetration Testing Tool lifecycle path used by Skill 033 security branches:
  - `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`
- Define and validate the request-readiness remediation orchestration path so underconfigured existing/generated REST/SOAP client tests and DB Tools can be repaired before validation or later orchestration continues:
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
- Maintain and refine the validated v1 single constrained REST Client lifecycle path that uses shell-first YAML promotion for constrained binding and REST Client `GET/PUT/GET` for constrained JSON request payload normalization, while keeping broader orchestration work out of scope:
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
- Define and validate the top-level `Change Advisor` orchestration path so one-source-spec bulk constrained REST Client OpenAPI refactor work has a stable owner for intake, grouped review, copied-target sequencing, and completion reporting:
  - `docs/skills/composite-orchestration/skill-061-change-advisor-bulk-openapi-refactor.md`
- Validate the rewritten TestAssets-backed project bootstrap and repair flow so registry self-heal, active-project routing, environment-file naming, and sensitive-reference handling behave consistently:
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`
  - `TestAssets/project-index.yaml`
- Define and thread the canonical secret-reference consumption + write-time auth materialization boundary so bootstrap stays reference-oriented while supported runtime auth branches may resolve approved gitignored secrets and materialize them into authored `.tst` assets:
  - `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`
  - `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md`
  - `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
- Validate and extend the new API-preferred mutation owners for ordinary rename, generic tool lifecycle operations, and broader disabled-state mutation:
  - `docs/skills/structure/skill-068-rename-object.md`
  - `docs/skills/structure/skill-070-copy-tool.md`
  - `docs/skills/structure/skill-071-move-tool.md`
  - `docs/skills/structure/skill-072-delete-tool.md`
  - `docs/skills/structure/skill-074-set-disabled-state.md`
- Merged local-workspace architecture rewrite closeout is complete; deferred legacy platform cards `002-005` remain on disk only as non-routing historical compatibility references.
- Broaden Virtualize coverage beyond shared platform/policy surfaces.
- Convert remaining partially-defined validation/data-exchange cards into fully validated workflows:
  - `docs/skills/platform/skill-024-tst-create-from-raml.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`
- Continue refining contributor-facing canonical docs so `skill-index.md`, `backlog.md`, and workflow docs stay tightly aligned.
- Validate and tune the cross-cutting XPath scalar-normalization policy in mixed XML/JSON tool chains:
  - `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`

## Skill Status Registry
### 1) Platform / File Operations
- `docs/skills/platform/skill-family-server-file-lifecycle.md`
  - **State:** Defined
  - **Notes:** canonical local-first selection/anti-duplication surface for merged-workspace file lifecycle operations, root scoping, ambiguity handling, and escalation into Skill 001, Skill 006, or Skill 050 as needed.
- `docs/skills/platform/skill-001-shared-introspection.md`
  - **State:** Defined
  - **Notes:** canonical shared local asset-path resolution surface for `TestAssets/`, `VirtualAssets/`, and `ProvisioningAssets/`, with active-project-first lookup, likely-root-first search, and fail-closed ambiguity handling.
- `docs/skills/platform/skill-002-shared-file-transfer.md`
  - **State:** Deferred/Retired
  - **Notes:** deferred legacy compatibility/reference card only; removed from `skill-index.md` so it is no longer an operator-facing routing target.
- `docs/skills/platform/skill-003-server-copy.md`
  - **State:** Deferred/Retired
  - **Notes:** deferred legacy compatibility/reference card only; removed from `skill-index.md` so it is no longer an operator-facing routing target.
- `docs/skills/platform/skill-004-server-rename.md`
  - **State:** Deferred/Retired
  - **Notes:** deferred legacy compatibility/reference card only; removed from `skill-index.md` so it is no longer an operator-facing routing target.
- `docs/skills/platform/skill-005-server-delete.md`
  - **State:** Deferred/Retired
  - **Notes:** deferred legacy compatibility/reference card only; removed from `skill-index.md` so it is no longer an operator-facing routing target.
- `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - **State:** Defined
  - **Notes:** narrow rollback-preserving local YAML edit envelope for explicitly authorized local YAML branches across merged-workspace asset types.

### 2) Asset Creation / Generation
- `docs/skills/platform/skill-021-tst-create-empty.md`
  - **State:** Validated
  - **Notes:** empty `.tst` create/reuse path is stable.
- `docs/skills/platform/skill-022-tst-create-from-openapi.md`
  - **State:** Validated
  - **Notes:** OpenAPI/Swagger generation path is validated and hardened for ambiguous write recovery. Current evidence shows attached external env files are not used to resolve `location.url=${...}` during create, but concrete-URL generation can emit matched external variable names into the generated `.tst` when the env file contains recognized OpenAPI/base-url variables (including prefixed names such as `PARABANK_SWAGGER` / `PARABANK_BASEURL`). Caller-owned post-generation normalization therefore depends on generated readback, not a fixed `${BASEURL}` assumption.
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
  - **Notes:** reviewed in wave 2 and aligned to the merged local-workspace model as a local-path-authoritative, YAML-evidence summary route.
- `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
  - **State:** Defined
  - **Notes:** foundational datasource discovery card; family-by-family validation still pending.
- `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - **State:** Defined
  - **Notes:** reviewed in wave 2 and aligned to the merged local-workspace model as a local-path-authoritative, YAML-only configuration/dataflow analysis route.

### 4) Structural Mutation
- `docs/skills/structure/skill-family-tst-object-manipulation.md`
  - **State:** Defined
  - **Notes:** architectural family/intent map for API-preferred object mutation routing, including rename, generic tool lifecycle operations, disabled-state mutation, and explicit YAML-exception boundaries.
- `docs/skills/structure/skill-068-rename-object.md`
  - **State:** Defined
  - **Notes:** atomic API-backed owner for in-place rename of supported suites and representative tool classes through class-specific `GET/PUT/GET`; created to stop ordinary rename drift into local YAML editing.
- `docs/skills/structure/skill-family-tool-lifecycle-operations.md`
  - **State:** Defined
  - **Notes:** routing family for generic tool copy/move/delete operations owned by `/v6/tools*`.
- `docs/skills/structure/skill-070-copy-tool.md`
  - **State:** Defined
  - **Notes:** atomic generic tool copy owner using `POST /v6/tools/copy` with explicit readback of the copied object.
- `docs/skills/structure/skill-071-move-tool.md`
  - **State:** Defined
  - **Notes:** atomic generic tool move owner using `POST /v6/tools/move` with readback confirmation of final parent placement.
- `docs/skills/structure/skill-072-delete-tool.md`
  - **State:** Defined
  - **Notes:** atomic generic tool delete owner using `DELETE /v6/tools` with absence verification and ambiguous-write reconciliation.
- `docs/skills/structure/skill-family-disabled-state-mutation.md`
  - **State:** Defined
  - **Notes:** broader cross-asset family for supported enable/disable work instead of treating disabled-state as tool-only lifecycle detail.
- `docs/skills/structure/skill-074-set-disabled-state.md`
  - **State:** Defined
  - **Notes:** atomic owner for supported disabled-state mutation via `PUT /v6/tools/disabled`, with v1 readback coverage for tools, test suites, and responder suites.
- `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
  - **State:** Defined
  - **Notes:** reviewed in wave 2 as a hybrid card: local `.tst` path/YAML anchor first, API runtime move branch only when runtime ids are resolved safely, with local YAML fallback through Skill 006.
- `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
  - **State:** Defined
  - **Notes:** broad suite lifecycle card exists; it now also serves as the structural owner used by cluster-oriented orchestrations such as Skill 065 for canonical suite skeleton creation and exact child-order realization.
- `docs/skills/structure/skill-055-testsuite-create-from-wsdl.md`
  - **State:** Validated
  - **Notes:** suite-level WSDL generation workflow validated for `disabled`, collision-aware `local_managed`, post-generation environment normalization into the original active environment, nested-parent placement readback, and a validated but non-default `reference_external` branch that can add referenced environment nodes without implicitly switching the active environment.
- `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
  - **State:** Defined
  - **Notes:** canonical v1 owner for SOAtest environment terminology, external project environment-file authoring, internal/referenced environment lifecycle, active-environment verification, local-to-external consolidation, shared generation-mode semantics across Skills 022-025, and caller-owned generated-local-variable insertion/inspection. Current OpenAPI evidence shows attached external env files do not resolve `location.url=${...}` during create, but concrete-URL generation can emit matched external variable names into the generated `.tst`, so callers must inspect both the env file and generated readback before normalizing.
- `docs/skills/structure/skill-047-generated-subset-prune.md`
  - **State:** Validated
  - **Notes:** generated-suite prune workflow is validated and reusable.

### 5) Client Tools
- `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
  - **State:** Validated
  - **Notes:** DB Tool lifecycle validated.
- `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - **State:** Validated
  - **Notes:** unconstrained REST Client creation/configuration is the current recommended path, including validated read-merge-write Basic-auth updates on existing clients through the native `httpAuthentication` subtree with readback confirmation of the persisted auth subtree and caller-owned concrete-host normalization of generated template clients to `${BASEURL}` plus relative path/query; approved gitignored secret references may now be resolved and materialized directly into supported Basic-auth writes, while schema-visible `ntlm`/`kerberos`/`digest` remain unvalidated.
- `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`
  - **State:** Defined
  - **Notes:** first-pass SOAP Client lifecycle card covering the API-exposed HTTP request/transport/misc surface; validated on scaffold/readback, WSDL-originated-compatible copy/update preservation, and `Response SOAP Envelope` output mapping, while full WSDL-tab parity and non-HTTP transport authoring remain pending.
- `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - **State:** Validated
  - **Notes:** validated v1 owner for single constrained REST Client lifecycle work; reviewed in wave 2 and clarified as a hybrid card with local `.tst`/YAML authority for asset structure and API-only entry for runtime ids, tool lifecycle routes, constrained JSON body normalization, and constrained-client Basic-auth updates through the native REST Client auth subtree, including approved secret-reference-backed plaintext auth materialization for supported branches. Future OAuth2/YAML-auth editing remains out of scope.
- `docs/skills/client-tools/skill-060-single-constrained-rest-client-openapi-refactor.md`
  - **State:** Defined
  - **Notes:** reviewed in wave 2 and aligned to the merged local-workspace model as a hybrid one-client refactor leaf: local `.tst` slice and copied-target structure first, Skill 050 only when runtime ids or API-normalized body branches are required.

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
  - **Notes:** JSON Data Bank card exists; runtime validation remains pending. The experimental lane now also depends on this card's ability to reuse and extend an existing producer-local databank for validation-only extracts rather than creating a sibling databank by default.
- `docs/skills/data-exchange/skill-075-data-generator-lifecycle.md`
  - **State:** Defined
  - **Notes:** full CRUD lifecycle card for Data Generator Tools defined from the local OpenAPI schema plus a local `.tst` exemplar; string-generator `&` / `#` semantics are documented, while runtime create/update validation is still pending.

### 8) Execution Diagnostics
- `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - **State:** Validated
  - **Notes:** canonical execution payload/results/traffic baseline, including the documented constraint that `testNames` is coarse and should be secondary to structural isolation in richer orchestrations.
- `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - **State:** Validated
  - **Notes:** focused failure diagnosis from runtime traffic is validated.

### 9) Cross-Cutting Policy
- `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - **State:** Defined
  - **Notes:** policy card for XPath-over-JSON selector semantics.
- `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - **State:** Defined
  - **Notes:** deterministic naming policy is established, with a generic pattern plus a branch-aware extension for clustered orchestrations such as Skill 065.
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - **State:** Validated
  - **Notes:** core chaining semantics are established and referenced broadly.
- `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - **State:** Validated
  - **Notes:** canonical parent/output mapping authority.
- `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - **State:** Defined
  - **Notes:** policy-level card for tool-managed request headers and native auth ownership; REST Client Basic-auth preference for the native auth subtree over literal `Authorization` header injection is now validated.
- `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`
  - **State:** Defined
  - **Notes:** canonical boundary card separating bootstrap secret-reference storage from later runtime consumption; approved gitignored secret refs may be resolved for exploration/execution and materialized into supported `.tst` auth writes, while source-control protection after generation remains the user's responsibility.
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - **State:** Validated
  - **Notes:** read-merge-write policy promoted from observed tool PUT behavior.
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - **State:** Defined
  - **Notes:** canonical API-branch preflight policy for runtime-object discovery, semantic mutation, generation, execution, and diagnostics; explicitly out of scope for pure local filesystem work.
- `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`
  - **State:** Defined
  - **Notes:** policy-level counterpart to Skill 049 for non-tool objects.
- `docs/skills/cross-cutting/skill-054-xpath-scalar-extraction-normalization.md`
  - **State:** Defined
  - **Notes:** centralizes XML scalar-selector `/text()` normalization guidance with explicit JSON boundary behavior.
- `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
  - **State:** In progress
  - **Notes:** additive experimental owner for bounded direct endpoint exploration, cluster-aware stateful ledgers, ordered authoring paths, prerequisite-expansion metadata, exploration-evidence legitimacy, and advisory negative-opportunity plus security-targeting hints with bounded extra discovery only when that improves downstream negative/security accounting inside the explicit opt-in live-exploration lane only.

### 10) Composite Orchestration
- `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`
  - **State:** In progress
  - **Notes:** canonical underspecified-intent entry point for service-test authoring; now uses staged intake, an interactive-only approval artifact, a true no-interrupt autonomous execution posture with default payload fill plus blocker-only clarifications, preferred wording for mandatory intake questions, dynamic standard-negative family planning with a cap-of-five ceiling and a richer protocol/schema/state candidate-family set, separate interactive security intake, an always-on `033 -> 058 -> negatives/security -> combined 057 validation pass` sequencing model, post-completion DB/full-run follow-up options, an execution-efficiency rule that excludes copied security suites from interim calibration/traffic/validation runs unless the user explicitly wants an end-of-workflow full `.tst` execution, and caller-owned batched blocker presentation when Skill 058 cannot finish bounded autonomous remediation.
- `docs/skills/composite-orchestration/skill-056-single-client-authoring-intent-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for one-client authoring intent across endpoint-only and contract-informed REST/SOAP flows; now preserves interactive versus autonomous posture through the one-client branch, defaults unresolved autonomous host placement to a minimal new `.tst` under the active project root when project context exists, delegates environment mechanics to Skill 064 when needed, provides preferred wording for mandatory one-client questions, and routes REST/SOAP execution to the smallest matching lifecycle leaf rather than broad generation cards.
- `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for adding validation to existing or newly generated tests and DB Tool resultsets, with live-runtime-evidence gating, multi-signal readiness fail-closed behavior, an explicit remediation-approval-vs-validation-approval boundary, combined happy-path-plus-standard-negative plan/approval by default when both slices are ready, an empty-response-payload no-tool exception for REST Client validation, happy-path schema-plus-content bundle defaults for API responses, default conservative validation for standard negatives, explicit security-branch no-tool exception handling, and response-vs-database orchestration.
- `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - **State:** Defined
  - **Notes:** branch-specific orchestration card for detecting and repairing underconfigured existing/generated REST/SOAP client tests and DB Tools before validation or later orchestration proceeds; now consults the active project record ahead of weaker inference sources, preserves inherited autonomous posture from Skill 033, uses a bounded one-synthesis-plus-one-evidence-driven-correction remediation loop per target, returns unresolved targets as an aggregated blocker set instead of interrupting inline, and keeps autonomous live-service probing and DB-side strategy out of scope unless explicitly approved.
- `docs/skills/composite-orchestration/skill-061-change-advisor-bulk-openapi-refactor.md`
  - **State:** Defined
  - **Notes:** reviewed in wave 2 and aligned to the merged local-workspace model as a hybrid composite: local source/copy `.tst` authority for analysis and copied-target sequencing, grouped review ownership at the orchestration layer, and Skill 050 only for delegated runtime-id/API-normalized write branches.
- `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md`
  - **State:** In progress
  - **Notes:** additive explicit opt-in top-level owner for the experimental exploration-first REST/OpenAPI broad-authoring lane that now centers on a disabled generated template suite, sibling `AI Generated Tests`, post-generation `BASEURL` template normalization before later copies inherit parameterized endpoints, cluster-local authored structure, stateful happy-path sequencing, phase-eligibility-aware required-phase accounting, autonomous continuation until no next safe in-scope action remains, required per-slice negative-accounting, required per-operation security-accounting with lifecycle-preferred source selection, final cleanup of empty happy-path child suites, explicit Skill 65 -> 67 validation handoff, narrow deferred handling, and Skill 074-focused verification sandboxing.
- `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`
  - **State:** In progress
  - **Notes:** additive exploration-backed validation owner for the experimental lane; preserves stable leaf mechanics while replacing the stable pre-attachment baseline run with trusted exploration evidence plus a Skill 65-owned handoff packet, databank-reuse-first enrichment, posture inheritance from Skill 65, and focused post-attachment verification.
- `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`
  - **State:** Defined
  - **Notes:** canonical session-start owner for matching, loading, creating, repairing, and updating durable application project context under `TestAssets/`; encodes deterministic registry matching, active-project routing, canonical project environment-file locations and references, environment-aware service-definition intake with multi-environment disambiguation, service-definition readback plus BASE_URL confirmation, explicit sensitive-data storage choices with a gitignored `.soavirt/projects/<slug>/secrets.env` default, the downstream reference-only bootstrap boundary for secrets, and the progressive load order `TestAssets/project-index.yaml -> TestAssets/<slug>/<slug>.yaml -> referenced files`.

### 11) Security Testing
- `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`
  - **State:** Defined
  - **Notes:** atomic lifecycle owner for Penetration Testing Tool create/read/update work under REST Client `Traffic Object`, including the explicit security-testing exception to normal business-validation chaining rules; operation-level breadth and lifecycle-preferred seed selection remain caller-owned by the orchestrating workflow.

## Coverage Gaps / Planned New Cards
- Newly added operation-centric owners close the main rename/copy/move/delete/disabled-state routing gap, but broader validation remains desirable for:
  - additional tool classes beyond the current rename matrix
  - direct test-node disabled-state readback ownership
  - any future responder/test rename owner if product workflows need it
  - explicit reorder cards/overlays
- Virtualize-specific skill coverage remains shallow beyond shared platform/policy surfaces.
- Additional datasource-family validation evidence is still desirable for Skill 051 and downstream datasource-aware workflows.
- Repeated-execution response-volatility profiling for validation-bundle selection remains future work rather than part of the v1 validation-enrichment orchestration.
- Higher-scope constrained REST Client work remains future research, including broader operation-retargeting/multi-client orchestration beyond the validated shell-promotion workflow and non-JSON constrained body modes.
- OAuth2 and other non-Basic REST Client auth authoring paths remain future research, especially where YAML-safe-editing or non-native auth/header ownership decisions would be required.
- Future architectural refactor candidate: evaluate whether operator-facing validation/data-exchange lifecycle cards (for example Skills 010, 028, 029, and 031) should eventually separate user-facing lifecycle orchestration from a narrower precomputed action-application contract so higher-order workflows can reuse them without reopening tool-family selection, approval, or baseline-planning logic.
- Related orchestration follow-up: analyze why Skill 033 still feels awkward/unreliable in execution and whether some of that friction comes from missing lower-layer action-contract boundaries rather than only from top-level intake/orchestration complexity.
- Future orchestration/leaf candidate: add a narrower negative-test workflow that can take a specified existing happy-path test and generate standard/security negative coverage for that one target without reopening full Skill 033 intake.

## Deferred / Retired
- REST Client constrained creation cards (`Skills 026/027`)
  - **State:** Deferred/Retired
  - **Notes:** their original narrow scope is superseded by the validated single-client constrained lifecycle coverage in `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`; unconstrained REST Client creation still routes to `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.

## Virtualize Future Track
- Deeper `.pva` lifecycle and deployment-aware workflows.
- Virtualize-specific validation of shared policies already written at the cross-cutting/platform layers.
- Additional operator-facing routing once Virtualize coverage grows beyond the current shared foundation.
