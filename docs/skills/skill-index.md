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
### 0) Project Context / Session Bootstrap
Route to `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md` when the user wants to:
- tell the agent what project they are working on at session start
- load or resume stored project context for a known project
- register a new project for future sessions
- update saved project facts such as environments, service-definition references, business data, or durable notes
- persist reusable project-specific context before any further task work in the session

Preferred prompts:
- `I'm working on the Parabank project`
- `load the project context for Loan Service`
- `this is a new project`
- `update the stored project info for Parabank`
- `remember this project for future sessions`

Workflow note:
- `docs/workflow/agent-workflow.md` invokes this skill automatically immediately after the AGENTS/bootstrap required reading only when startup classification resolves to operator bootstrap (explicit project/operator starts or the bare `Follow @AGENTS.md` default).
- Explicit contributor-start prompts skip automatic Skill 063 entry and continue in contributor mode unless the user also asks to load/store/update project context or later shifts into project-aware runtime work.
- This file routes to Skill 063; the card owns the exact bootstrap interview, matching, repair, and persistence procedure.
- Once Skill 063 has resolved an active project, later direct routing to smaller cards still inherits the workflow-level project-context contract for placement, source/value selection, validation priorities, and environment-sensitive behavior.

### 0.5) SOAtest Environment Mechanics
Route to `docs/skills/structure/skill-064-soatest-environment-lifecycle.md` when the user wants to:
- inspect or explain internal SOAtest environments versus external project environment files
- create or update project-scoped external `.env` / `.envs` files
- create, inspect, update, attach, or remove internal/referenced environment nodes in a `.tst`
- merge/fold values between an internal environment and the canonical project external environment file
- make a referenced environment node active, verify active-environment state, or safely remove redundant local environments after environment consolidation
- choose or verify `disabled`, `local_managed`, or `reference_external` environment strategy for service-definition-driven generation

Preferred prompts:
- `attach this project environment file to this tst`
- `take these internal environment values and fold them into the external project env file`
- `make the referenced environment active and remove the internal one`
- `inspect the environments in this tst`
- `set up the environment mode for generation from this OpenAPI`

### A) Service-Test Authoring
Route to `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` when the user wants to:
- create new service tests
- generate tests from OpenAPI/WSDL/RAML/XSD but key routing dimensions are still unresolved
- turn an endpoint/service definition into tests but has not yet clarified whether they want one client, a new `.tst`, or tests added into an existing `.tst`
- add generated tests into an existing `.tst` but still needs intake/branching help
- gather scope and plan before test authoring
- resolve whether a contract is the generation source or only planning input

Use Skill 033 as the canonical route for underspecified service-test authoring prompts, not as the mandatory route for already-specific direct requests.
#### Experimental exploration-first routing
Route to `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md` when the user explicitly wants the additive experimental broad-authoring lane, for example:
- they explicitly ask for the experimental branch
- they explicitly ask for exploration-first or live-probing broad authoring
- they explicitly want direct endpoint research before SOAtest writes
- the source is REST/OpenAPI and the request is still broad authoring rather than a narrow lifecycle action

Preferred prompts:
- `use the experimental branch for this OpenAPI`
- `try the exploration-first approach for this service`
- `use live endpoint exploration before authoring the tests`
- `I want the experimental broad authoring lane`

Boundary note:
- explicit opt-in is required
- keep Skill 033 as the default broad-authoring route when that opt-in signal is absent
- do not treat Skill 067 as a first-choice direct route in the initial rollout; the operator-facing experimental entry point is Skill 065
#### Direct generation eligibility gate (required)
Route directly to generation cards (Skills 022-025) only when all of the following are true:
- the user explicitly requests generation/create action from a service definition (OpenAPI/WSDL/RAML/XSD),
- the user explicitly commits to a **new `.tst`** output (for example, `create a new .tst from this OpenAPI`),
- the prompt is not primarily help-style or planning-style intake (for example, `help me create tests for this service definition`),
- no routing-critical dimensions remain unresolved (for example new vs existing `.tst` vs one-client intent).
If any condition is not met, start with Skill 033 and clarify.
#### Direct validation-enrichment routing
Route to `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md` when the user wants to:
- add validations to existing tests in a `.tst`
- enrich generated tests with schema validation and/or content validation
- add assertions or diff-style validation to DB Tool resultsets
- compare service responses with DB results for consistency
- add validation to a known set of happy-path or negative tests without re-opening broad authoring intake

Preferred prompts:
- `I have these tests in this .tst and want to add validations`
- `add schema validation to these happy-path tests`
- `enrich these generated tests with assertions`
- `add an XML Assertor to this DB Tool resultset`
- `compare these response tests with the DB`
#### Direct request-readiness remediation routing
Route to `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md` when the user wants to:
- help configure existing tests whose requests still look default or incomplete
- finish setting up request data for existing REST/SOAP client tests
- repair underconfigured DB Tools whose connection/query settings are still incomplete
- repair request-readiness before validation or later authoring steps continue
- remediate multiple existing tests that are failing because the requests are not configured yet

Preferred prompts:
- `help me configure these tests`
- `these clients still need request data`
- `this DB Tool still needs its connection/query configured`
- `these tests look default and need to be configured`
- `fix these tests so they are ready before validation`
#### Direct constrained REST Client lifecycle routing
Route to `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md` when the user wants to:
- create one OpenAPI-constrained REST Client from scratch through the validated shell-promotion workflow
- edit request values on one existing constrained REST Client without changing the selected operation
- edit JSON request payload values on one constrained `Form JSON` REST Client through the REST Client API normalization path
- copy or delete one known constrained REST Client
- use the YAML path for constrained promotion/resource/config edits and the REST Client API path for constrained JSON request bodies on one target client

Preferred prompts:
- `create one constrained REST client for this OpenAPI operation`
- `edit this constrained REST client's query parameters`
- `update this constrained REST client's JSON body payload`
- `copy this swagger-backed REST client`
- `delete this constrained REST client`

#### Direct constrained REST Client refactor routing
Route to `docs/skills/client-tools/skill-060-single-constrained-rest-client-openapi-refactor.md` when the user wants to:
- refactor one known constrained REST Client from one confirmed OpenAPI contract/version/location to another
- analyze or apply one-client source-to-target OpenAPI refactor work without broad multi-client orchestration
- repair one constrained REST Client subtree plus its supported downstream JSON validation/data-exchange tools as one unit

Preferred prompts:
- `refactor this constrained REST client from v1 to v2`
- `retarget this one constrained REST client to the new OpenAPI`
- `analyze how this one constrained REST client would map to the new spec`
- `apply the one-client openapi refactor for this constrained test`

#### Direct bulk OpenAPI refactor routing
Route to `docs/skills/composite-orchestration/skill-061-change-advisor-bulk-openapi-refactor.md` when the user wants to:
- refactor an existing `.tst` from one confirmed OpenAPI contract/version/location to another across all matching constrained REST Clients in one source-spec slice
- analyze or apply bulk source-to-target OpenAPI refactor work on a copied target asset
- review grouped migration decisions across multiple constrained REST Client targets before writes begin

Preferred prompts:
- `refactor this tst from v1 to v2`
- `run change advisor on this existing tst`
- `analyze this tst for bulk openapi migration`
- `migrate all constrained clients in this tst from the old spec to the new spec`

#### Direct single-client routing
Route to `docs/skills/composite-orchestration/skill-056-single-client-authoring-intent-orchestration.md` when the user wants to:
- create one REST Client or one SOAP Client
- create one operation-focused test rather than broad generated coverage
- use OpenAPI/WSDL as planning input for one client
- create a quick endpoint-focused client/test rather than a generated suite/file

Preferred prompts:
- `create one REST client for this endpoint`
- `make one SOAP client for this operation`
- `I only want one operation from this OpenAPI`
- `use this WSDL to help create a single SOAP Client`

#### Direct generation routing
Route directly to the smallest matching generation workflow only when the prompt passes the direct generation eligibility gate above:
- new `.tst` from OpenAPI -> `docs/skills/platform/skill-022-tst-create-from-openapi.md`
- new `.tst` from WSDL -> `docs/skills/platform/skill-023-tst-create-from-wsdl.md`
- new `.tst` from RAML -> `docs/skills/platform/skill-024-tst-create-from-raml.md`
- new `.tst` from XSD -> `docs/skills/platform/skill-025-tst-create-from-xsd.md`
- WSDL-generated suite added into an existing `.tst` -> `docs/skills/structure/skill-055-testsuite-create-from-wsdl.md`

Tie-break rule:
- If the request is vague, use Skill 033.
- If the user explicitly opts into the experimental exploration-first broad-authoring lane for REST/OpenAPI, use Skill 065.
- If the request is help-style or outcome-level (for example `help me create tests for this OpenAPI`) and does not explicitly request a **new `.tst`**, use Skill 033.
- If the request is already clearly validation-enrichment intent on existing/generated tests, use Skill 057.
- If the request is already clearly request-readiness/configuration-remediation intent on existing/generated tests, use Skill 058.
- If the request is already clearly single constrained REST Client lifecycle work within the validated shell/promotion boundary, use Skill 059.
- If the request is already clearly bulk one-source-spec constrained REST Client OpenAPI refactor work on an existing `.tst`, use Skill 061.
- If the request is already clearly one-client constrained REST Client source-to-target OpenAPI refactor work, use Skill 060.
- If the request is already clearly one-client intent, use Skill 056.
- If the request is already clearly direct generation intent, explicitly targets creation of a new `.tst` from a service definition, and the capability exists as a dedicated card, bypass Skill 033 and route directly to that card.

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

### C) Operation-Centric Mutation Routing
Use this registry before falling back to family-level navigation for object/tool mutation prompts.

#### Route to `docs/skills/structure/skill-068-rename-object.md`
Use when the user wants to rename one existing supported suite or tool and no broader configuration change is in scope.

Preferred prompts:
- `rename this tool`
- `rename this suite`
- `change the name of this rest client`

#### Route to generic tool lifecycle leaves
Use when the user wants a pure tool copy, move, or delete and no broader tool-family configuration work is in scope.

Preferred prompts:
- `copy this tool into that suite`
- `move this diff tool`
- `delete this validator`

Routes:
- copy -> `docs/skills/structure/skill-070-copy-tool.md`
- move -> `docs/skills/structure/skill-071-move-tool.md`
- delete -> `docs/skills/structure/skill-072-delete-tool.md`

#### Route to `docs/skills/structure/skill-074-set-disabled-state.md`
Use when the user wants to enable or disable one supported existing asset without broader configuration changes.

Preferred prompts:
- `disable this tool`
- `enable this suite`
- `turn this responder suite back on`

#### Tie-Break Rule (Required)
- If the prompt is a pure operation-only request, prefer the smallest operation-centric owner above.
- If the prompt is already a broader class-specific create/update/configuration workflow, keep the domain-specific lifecycle card as the primary owner.
- YAML-authorized exception cards are not fallback routes for these ordinary mutation prompts.

## Primary Navigation by Skill Family

### 1) Platform / Local File Operations
Use for local merged-workspace asset targeting, local file lifecycle guidance, and helper-only YAML rollback that is entered only from an explicitly authorized owning skill.
- `docs/skills/platform/skill-family-server-file-lifecycle.md`
- `docs/skills/platform/skill-001-shared-introspection.md`
- Helper only, not a first-choice routing target: `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`

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
Use for suite/object creation, rename, generic tool lifecycle operations, disabled-state changes, movement, reordering, and subset pruning.

- `docs/skills/structure/skill-family-tst-object-manipulation.md`
- `docs/skills/structure/skill-068-rename-object.md`
- `docs/skills/structure/skill-family-tool-lifecycle-operations.md`
- `docs/skills/structure/skill-family-disabled-state-mutation.md`
- `docs/skills/structure/skill-008-datasource-type-targeted-move.md`
- `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
- `docs/skills/structure/skill-055-testsuite-create-from-wsdl.md`
- `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
- `docs/skills/structure/skill-047-generated-subset-prune.md`

### 5) Client Tools
Use for client-tool lifecycle and configuration.

- `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
- `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
- `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`
- `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
- `docs/skills/client-tools/skill-060-single-constrained-rest-client-openapi-refactor.md`

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
- `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`

#### Capability / Compatibility
- `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`

### 10) Composite Orchestration
Use for multi-step conversational intake and cross-skill planning/execution.

- `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`
- `docs/skills/composite-orchestration/skill-056-single-client-authoring-intent-orchestration.md`
- `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
- `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
- `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md`
- `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`
- `docs/skills/composite-orchestration/skill-061-change-advisor-bulk-openapi-refactor.md`
- `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`

### 11) Security Testing
Use for security-testing tool lifecycle and security-branch attachment rules.

- `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`

## Selection Heuristic
1. For each new user prompt, re-run this routing heuristic before reusing any previously selected target card.
2. If `AGENTS.md`/bootstrap has been loaded for the current session, startup classification resolved to operator bootstrap, and Skill 063 has not yet completed its bootstrap, use Skill 063 first. Skip this automatic entry for explicit contributor-start sessions unless the user later asks to load/store/update project context or begins project-aware runtime work. If the user explicitly asks to load/store/update project context later in the session, use Skill 063 again.
2a. After Skill 063 has completed, treat the active project as in-scope runtime state for later direct routes whenever the branch still involves placement defaults, request/config synthesis, source resolution, validation priorities, or environment-sensitive behavior.
3. Determine whether the request is:
   - authoring/generation
   - read-only analysis
   - structural mutation
   - tool lifecycle/configuration
   - execution diagnostics
4. Apply any explicit routing registry above.
5. If no explicit routing rule applies, choose the smallest matching skill family from the Primary Navigation section.
6. Load the target card and its declared dependencies.
7. For server-API-mediated Parasoft requests, also apply the workflow-mandated runtime prelude from `docs/workflow/agent-workflow.md`; local file-backed requests should stay local-first and enter Skill 050 only if the branch later requires API interaction.
8. Avoid loading unrelated cards.

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
