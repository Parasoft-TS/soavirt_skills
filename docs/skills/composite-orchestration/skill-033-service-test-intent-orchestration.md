# Skill 033: Service-Test Intent Orchestration (Composite)

## 1) Skill Name
Gather user testing intent and orchestrate service-test authoring across atomic skills.

## 2) Objective
Convert underspecified requests (for example "create tests for this service") into a confirmed, executable test-authoring plan before any write operations are performed.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting scope and preference input.

## 3) Scope
- In scope:
  - intent intake for endpoint-only or service-definition-driven requests
  - confidence-gated required-input resolution
  - operation/contract summarization for user scope decisions
  - progressive question bundles (coverage, negative tests, validators, DB validation)
  - deterministic orchestration plan mapped to existing atomic skills
  - staged execution with verification checkpoints
- Out of scope:
  - replacing endpoint-specific behavior in atomic cards
  - domain-specific business test recipes as mandatory defaults
  - autonomous write execution without explicit user confirmation

## 4) Inputs
- Required:
  - user objective to create service tests
  - at least one initial anchor:
    - service endpoint, or
    - service definition/schema location (user-provided URL or file-system path)
- Conditionally required before first write:
  - target `.tst` file name (or explicit approval to use deterministic auto-name)
  - for write methods (`POST`/`PUT`/`PATCH`): request-body materialization mode and approved concrete payload for each happy-path operation that requires a body
- Optional:
  - preferred service-definition format (`openapi`, `wsdl`, `raml`, `xsd`)
  - preferred operation scope (all/subset)
  - validation strategy preferences
  - DB validation intent and known connection/query constraints

## 5) Preconditions
- API reachable and authenticated for all downstream atomic skills.
- Parent workspace location for new/updated assets is resolvable.
- User intent has reached minimum clarity threshold (see Section 6.4 confidence gate).

## 6) Procedure
### 6.1 Intake Classification
1. Classify request shape:
   - endpoint-only request,
   - service-definition-provided request,
   - mixed/ambiguous request.
2. Resolve missing required inputs by precedence:
   - explicit values in current prompt,
   - values confirmed earlier in current session,
   - relevant environment variables.

### 6.1.1 API Capability Preflight (Required)
2.1 Before any write operation, run branch-aware capability preflight (Skill 050):
  - verify base API reachability with `GET /v6/children`,
  - probe endpoint-method pairs used by the selected branch,
  - classify unsupported method routes (`405`) and select documented fallbacks.
2.2 Do not continue with optimistic endpoint assumptions when preflight has identified a fallback path.

### 6.2 Source Strategy Branch
3. If service definition/schema is missing or ambiguous, ask targeted question and wait.
4. If user has service definition/schema, confirm format and location type (URL vs file path).
5. If user only has endpoint and no service definition:
   - offer two paths:
     - quick-start endpoint tests (Skill 020 path), or
     - pause and supply service definition for broader generation (Skills 022-025 path).

### 6.2.1 Source Normalization (Internal, Not User-Facing)
6. Normalize user-provided source to API-compatible location fields:
   - URL input -> `location.url`
   - local/relative file path -> convert to workspace-accessible location:
     - prefer uploading/resolving to workspace file and using `location.id`, or
     - when runtime supports direct file URI, convert to canonical file URI form and validate reachability.
7. Do not ask users for `location.id` terminology directly unless they explicitly request API-level detail.
8. For deterministic parsing and traceability, materialize a transient source snapshot under `work/` before generation:
   - fetch/download the service definition using a deterministic transport (for example `curl`/HTTP client),
   - avoid UI/browser save dialogs,
   - parse operation/response metadata from the transient copy used for planning.

### 6.3 Scope Discovery (Progressive Sequence)
8. Ask concise high-impact intake questions one at a time, waiting for the user's answer before asking the next:
   - Coverage: all operations vs subset.
   - Depth: happy-path only vs include negative tests.
   - Validation: minimal (expected response code baked into client tool) vs explicit validator/assertor/diff stack.
   - Data consistency: include DB-backed validation or not.
9. Ask advanced follow-ups only for selected branches, one question at a time:
   - Negative scope: spec-defined non-2xx only vs additional AI-suggested error cases.
   - Validation scope: user-specified checks vs AI-proposed baseline then review.
   - DB scope: connection readiness, candidate SQL strategy, correlation keys.

### 6.3.2 Mandatory Artifact and Write-Request Gates
9.1 Before any create/copy/write action, resolve target test-asset naming:
  - ask the user for desired `.tst` file name, or
  - ask for explicit approval to use deterministic auto-name.
9.2 For write methods (`POST`/`PUT`/`PATCH`), resolve request payload strategy before creating tests:
  - user provides payload,
  - AI proposes payload and user approves/edits,
  - AI fills autonomously only when the user explicitly opts in.
9.3 Treat payload as unresolved until approval is explicit:
  - if payload is AI-proposed, print a compact preview and ask for approve/edit,
  - do not write REST Client body fields while payload approval is unresolved.
9.4 Echo the resolved request blueprint for each happy-path write test before writes:
  - method + URL + key query/path parameters,
  - payload source (`user-provided` or `AI-proposed-approved`),
  - target `.tst` file name.

### 6.3.1 Conversational Intake UX (Required)
10. Prefer conversational intake over forced option selectors:
  - ask short natural-language questions,
  - accept free-text responses,
  - interpret user phrasing into normalized plan values and echo interpretation for confirmation.
  - do not solicit multiple unresolved intake questions in a single prompt.
11. For subset selection, always print a human-readable operation catalog before asking the user to choose:
  - show method + path (+ short summary when available),
  - include a few recommended presets (for example core smoke, transactions, customers),
  - allow free-text selection (for example "all /customers endpoints", "just login and transfer").
12. Resolve conversational subset responses with deterministic matching and transparent echo:
  - support exact path, prefix group (for example `/customers`), comma lists, and keyword matches,
  - print the resolved operation list before execution,
  - if confidence is low, ask one clarifying question; otherwise continue.

### 6.4 Confidence Gate + Plan Confirmation
13. Build a human-readable execution plan summary:
   - selected operation scope,
   - planned test count profile,
   - planned tool chain per operation/profile,
  - target `.tst` file name,
  - write-request materialization summary (including payload source for `POST`/`PUT`/`PATCH`),
   - assumptions/unknowns requiring confirmation.
14. Require explicit user confirmation before writes.
15. If confirmation is partial, execute only confirmed slices.

### 6.5 Orchestration Mapping (Examples)
16. Map confirmed plan to atomic skills:
   - Service-definition generation:
     - OpenAPI -> Skill 022
     - WSDL -> Skill 023
     - RAML -> Skill 024
     - XSD -> Skill 025
   - Endpoint-only bootstrap:
     - REST client baseline -> Skill 020
   - Validation layer:
     - JSON/XML Validator -> Skills 029/030
     - JSON/XML Assertor -> Skills 010/016
     - Diff Tool -> Skill 031
   - Data extraction/correlation:
     - JSON/XML Data Bank -> Skills 028/019
   - DB validation:
     - DB Tool lifecycle -> Skill 015
   - Runtime verification/diagnostics:
     - execution triad -> Skills 012/014

### 6.6 Scale Control
17. For large services, default to phased rollout unless user requests full breadth:
   - phase by tag/group/domain,
   - phase by top-N operations,
   - phase by profile (happy-path first, then negatives, then DB).

### 6.6.1 Request Parameter Discovery and Materialization
After generation and subset pruning (rule 15), inspect the generated happy-path REST Clients for configurable request parameters beyond path parameters already resolved during intake.

**Discovery scope** (cross-reference each in-scope REST Client config with the service definition):
- path parameters not yet materialized from intake,
- required query parameters,
- optional query parameters,
- request body fields for write operations (POST/PUT/PATCH) — distinguish required vs optional.

**Materialization modes** — present the user with a per-operation parameter summary and offer:
- a) **AI proposes, user confirms** (recommended default): inspect the service definition for `example` values, `default` values, type constraints, and enum options; propose concrete values for every unconfigured parameter; present the proposals for the user to accept, override, or skip (for optional params).
- b) **User provides all**: walk through each REST Client that has unconfigured parameters and ask the user to supply values.
- c) **AI fills autonomously**: fill from spec examples or generated reasonable defaults without confirmation. Offer this mode only when the user explicitly opts in.

**Rules:**
- Always surface required parameters — do not leave required parameters unconfigured or silently filled.
- Flag optional parameters but allow the user to skip them (leave unconfigured).
- Apply confirmed values to happy-path REST Clients before creating negative variants, since negatives derive from the concretized happy baseline (rule 26).
- For request body operations, present the body schema structure with required/optional annotations so the user understands what fields exist.
- When AI proposes values, cite the source (for example `example` from spec, type-based generation, enum first-value) so the user can judge quality.

### 6.7 Execution Hardening Rules (Required)
14.5 Enforce preflight route selection for write-phase calls:
  - if preflight marked a read path unsupported for the current server, use fallback route immediately,
  - do not retry unsupported endpoint-method pairs in the same run unless server context changed.
14.6 Enforce ambiguous-write reconciliation before retry (required):
  - for any write with ambiguous command output, run deterministic readback reconciliation before deciding retry,
  - treat confirmed post-write state as success and continue orchestration,
  - for Skill 022 create branches, if same-name `.tst` is found after ambiguous output, do not re-POST create; follow reuse/delete-and-recreate/new-name handling.

15. Enforce subset pruning immediately after generation when user requested subset scope:
  - remove non-selected operation suites/tests before adding variants/tooling,
  - prune candidates must be filtered to direct children where `type=testSuite`; never delete non-suite nodes (for example `Environment`),
  - do not proceed with full-definition residue still present.
16. Use deterministic variant naming (non-cumulative labels):
  - `<operation> [happy]`
  - `<operation> [neg-spec-default]`
  - `<operation> [neg-ai-invalid-input]`
  - `<operation> [neg-ai-missing-resource]`
  - negative variants must never inherit `[happy]` in the final name.
  - `neg-spec-default` should represent the primary contract-driven negative for the operation (not just relaxed expected codes).
16.1 Negative request differentiation guard (required):
  - negative variants must differ from happy baseline at request level for the intended failure mode (for example path/query/payload mutation), not only expected-code settings,
  - after mutation, verify each negative request fingerprint differs from happy fingerprint for AI negatives and for `neg-spec-default` when a deterministic spec-driven mutation exists,
  - if mutation is a no-op (request unchanged), stop and re-derive failure-mode mutation before execution.
16.2 Negative selection examples (concrete defaults):
  - Path-parameter operations:
    - `neg-spec-default`: use a contract-invalid path value if schema/pattern/min-max constraints exist.
    - `neg-ai-invalid-input`: malformed id/token (for example `INVALID_ID`).
    - `neg-ai-missing-resource`: syntactically valid but likely absent id (for example `99999999`).
  - Required query parameters:
    - `neg-spec-default`: remove one required query parameter or use out-of-range value from contract constraints.
  - Request body operations:
    - `neg-spec-default`: omit one required field or violate enum/type constraint from schema.
    - `neg-ai-invalid-input`: malformed field value (wrong type/format).
  - Security-oriented negatives (authorized defensive testing only):
    - SQL injection probes in query/body fields that feed backend queries (for example quote-breaking and boolean-based payloads).
    - XSS probes in reflected/stored text fields (for example script/event-handler payload variants).
    - Oversized input stress (very large strings/arrays/blobs) to test length handling and potential overflow-like behavior.
    - Path traversal / file path manipulation probes where path-like inputs exist.
    - Header/cookie tampering probes (malformed auth/session tokens) where request security context is input-driven.
    - For each security negative, keep expected status/response assertions aligned to defensive behavior (reject/sanitize/error response), not exploit success.
  - Status-code expectations:
    - keep expected-code settings aligned with targeted failure mode, but do not treat expected-code edits as a substitute for request mutation.
17. Use fresh-id guard after any rename/copy/create operation:
  - re-read the parent children and resolve the current id by exact name before the next mutation,
  - never assume previous ids remain stable after name-changing writes.
18. Use parent resolution guard for copy/move:
  - source parent id must come from fresh readback relationships (for example `relationships.parentRel.id`),
  - never derive parent by string slicing id paths.
19. Negative expected-code calibration gate (required):
  - after creating negative variants, run a calibration execution before finalizing expected codes,
  - compare observed HTTP status codes against initially assumed expected codes,
  - update expected codes to match actual service behavior when spec does not define explicit error responses (for example spec only defines `default` response),
  - do not trust convention-based code assumptions (for example 400 for invalid input, 404 for missing resource) without runtime evidence,
  - log any surprising response codes (for example 200 for non-existent resource updates) as service behavior observations.
20. Use tool-update PUT safety guard (read-merge-write) for any pre-configured tool:
  - before editing an existing tool, read current config from its class-specific `GET` endpoint,
  - send a merged `PUT` payload that preserves fields not intentionally changed,
  - avoid sparse/name-only PUT payloads unless endpoint semantics are explicitly proven safe,
  - for REST Clients specifically, preserve `resource`, `header`, `httpOptions`, `payload`, and `misc` unless intentionally changing them,
  - load `cross-cutting/skill-049-tool-put-read-merge-write-policy.md` when updating existing tool instances.
20.1 Validation sequencing guard (required):
  - execution order for generated REST service tests must be:
    1) configure happy-path request data,
    2) clone/create negative variants from the finalized happy baseline,
    3) run negative calibration execution and finalize negative expected-code settings,
    4) attach validation tools to happy-path tests according to rules 21-23,
    5) attach validation tools to negative variants only when explicitly requested.
  - do not attach validation tools to happy tests before cloning negatives; this prevents copied validator/assertor/diff children from leaking into negatives by default.
  - if validation tools already exist on a happy-path parent before cloning, cloned negatives must remove inherited validation children unless user explicitly requested negative validation tooling.
20.2 Approved-plan completion guard (required):
  - after plan confirmation, do not interrupt execution with repeated proceed/stop prompts for known runtime failures unless a true blocker is hit,
  - treat business-level test failures as diagnostics outputs, not orchestration stop conditions,
  - continue remaining approved stages (for example negatives, calibration, validation tooling) and report residual failures in a final summary,
  - prompt the user only when a dependency is blocked (for example cannot resolve ids, cannot chain tool due invalid parent type, missing required source/schema with no safe fallback).
21. Validation-tool selection defaults for REST service requests:
  - run a pre-creation baseline execution for candidate happy tests and capture observed response media type/body shape,
  - capture baseline evidence from the executed REST Client output traffic (`/testExecutions/{id}/traffic` using the REST Client's `Traffic Viewer`), not from ad-hoc external endpoint calls,
  - choose validator/assertor families from observed runtime media type:
    - JSON response -> JSON Validator + (JSON Assertor or Diff Tool JSON mode),
    - XML response -> XML Validator + (XML Assertor or Diff Tool XML mode),
    - plain-text response -> do not attach JSON/XML tools by default; route to Diff Tool text mode workflow,
  - apply selection per endpoint/test independently; never propagate one endpoint's media type decision to other endpoints,
  - never attach JSON tools to non-JSON traffic or XML tools to non-XML traffic,
  - when schema validation is requested, configure with user-provided service-definition location and explicit message mapping,
  - when service-definition source is OpenAPI/Swagger and location is known, JSON happy-path validators must be configured as schema validators (not well-formedness-only) for JSON responses,
  - schema validator configuration must be applied per endpoint and must not be downgraded globally because another endpoint is non-JSON or fails tool creation,
  - for JSON responses, choose between Diff Tool JSON mode and JSON Assertor based on response data volatility:
    - mostly static response data -> prefer Diff Tool in JSON mode (simpler setup, full-response baseline comparison with minimal configuration),
    - significant dynamic/volatile data (timestamps, generated ids, session tokens) -> prefer JSON Assertor with targeted assertions on stable fields (Diff Tool would require many ignored differences, reducing its value since those dynamic elements are not tested anyway),
    - when unsure, start with Diff Tool and switch to JSON Assertor if the ignored-differences list grows large,
  - for XML responses, apply the same static/dynamic heuristic between Diff Tool XML mode and XML Assertor.
21.3 Per-endpoint validator and diff-mode integrity guard (required):
  - when rule 21.1 resolves a JSON or XML validator for an endpoint, creation of that validator is mandatory for that endpoint unless tool creation fails.
  - if validator creation fails for one endpoint, mark that endpoint validator as failed/skipped with reason and continue other endpoints; do not silently downgrade to Diff-only for that endpoint.
  - for each Diff Tool, set mode from that endpoint's observed response media type only (`json`/`xml`/`text`/`binary`) and verify readback for that tool id before moving to the next endpoint.
  - do not run bulk mode updates that can overwrite previously configured Diff Tool modes across endpoints.
21.1 Normative validation decision table (happy-path REST tests):

| Observed response media type | Schema validation requested | Volatility profile | Default tool set to attach |
|---|---|---|---|
| JSON | yes | low | JSON Validator + Diff Tool (JSON mode) |
| JSON | yes | high | JSON Validator + JSON Assertor |
| JSON | no | low | Diff Tool (JSON mode) |
| JSON | no | high | JSON Assertor |
| XML | yes | low | XML Validator + Diff Tool (XML mode) |
| XML | yes | high | XML Validator + XML Assertor |
| XML | no | low | Diff Tool (XML mode) |
| XML | no | high | XML Assertor |
| plain text | n/a | any | Diff Tool (text mode) |
| binary | n/a | any | Diff Tool (binary mode) |

21.2 Precedence order (required):
  - apply decision precedence in this order:
    1) explicit user override,
    2) media-type gate safety,
    3) normative table in rule 21.1,
    4) volatility fallback heuristic (only when table inputs are incomplete).
  - if rule 21.1 and any other narrative guidance conflict, rule 21.1 wins.
22. Chaining attachment rule:
  - discover outputs from the producer tool itself and select semantic response/results output,
  - never infer chain parent from sibling tool presence (for example `Traffic Viewer`).
23. Validation bundle expectation when user requests "validation tools":
  - include validator + comparison baseline by default only when media-type gate is satisfied (JSON/XML):
    - comparison tool is selected by rule 21.1 (assertor for high volatility, Diff Tool for low volatility),
  - for JSON/XML comparison tools, seed expected baseline content from the same test execution's response traffic payload that established media type,
  - for plain-text responses, default to Diff Tool text mode using baseline live response as expected value seed,
  - for plain-text responses, do not attach JSON/XML validator or assertor tools,
  - include Diff Tool when an expected/baseline artifact exists or user explicitly requests diff-based comparison,
  - de-duplicate validation intents per parent/output:
    - if both fallback routing and explicit diff request select Diff Tool for the same parent, create/update one Diff Tool only,
    - collapse repeated diff triggers into a single idempotent Skill 031 invocation.
24. Validation phase resilience (required):
  - if a validation tool fails to create or chain (for example 400 parent type error, output provider not found), log the failure and continue with the next validation tool in the plan,
  - do not let a single tool-creation failure abort the entire validation phase for a test or suite,
  - after the validation phase completes, report all skipped or failed tools with failure reasons so the user can address them,
  - retry failed tools only after root cause is identified and resolved (for example wrong parent id corrected).
24.1 Validation fallback policy (required):
  - if schema-based validator setup is requested but schema/message mapping inputs are missing or low-confidence, fall back to well-formedness validator mode for supported media type and continue,
  - do not block the approved orchestration plan solely due missing schema-mapping details unless the user explicitly required strict schema validation only.
24.2 Schema-validator strictness rule for known OpenAPI source (required):
  - if OpenAPI/Swagger source location is known from intake and observed response media type is JSON, do not use well-formedness fallback for that endpoint,
  - required behavior is explicit schema mapping (`autoDetectMessage=false`, direction/path/method/responseCode) and `validationType=validateAgainstSchema`,
  - if schema mapping cannot be resolved for one endpoint, mark that endpoint's validator as skipped/failed and continue other endpoints; do not silently downgrade successful JSON candidates.
25. Pre-execution URL readiness check (REST Client flows):
  - before invoking execution skills, sample targeted REST Clients and confirm `resource.literalText.fixed` is non-empty,
  - if empty URL is detected, stop execution and repair configuration before run.
26. Intake-parameter materialization for happy-path URLs:
  - when user provides concrete path/query values during intake (for example `customerId=12212`), apply them to happy-path REST Client resource templates before cloning negatives,
  - do not leave unresolved placeholders (for example `{customerId}`) in happy-path URLs when concrete intake values were provided,
  - derive negative variants from the concretized happy baseline so spec-default negatives retain realistic context.
27. Validator schema-mapping path rule (JSON/XML validators):
  - keep validator schema/message operation paths aligned to service-definition templates (for example `/customers/{customerId}`),
  - do not propagate concretized happy URL path values (for example `/customers/12212`) into validator definition mapping fields,
  - concretized runtime values are for request execution context; template paths are for schema/contract mapping context.

## 7) Validation
- No write operation occurs before explicit plan confirmation.
- No write operation occurs before `.tst` naming is resolved (explicit name or explicit auto-name approval).
- For `POST`/`PUT`/`PATCH` happy-path tests, no write occurs before payload source and payload content are explicitly approved.
- Every confirmed branch maps to explicit atomic skills.
- Produced plan is auditable (inputs, assumptions, selected scope).
- Runtime verification uses execution-triad skills for executed slices.
- Subset requests result in subset-only suites/tests after prune step.
- Variant names match deterministic naming rules without cumulative label drift.
- Multi-step write chains complete without stale-id or bad-parent errors.

## 8) Failure Modes
- Overlong questionnaire causes user drop-off.
- Ambiguous source format/location leads to incorrect generation branch.
- Scope explosion when user requests all operations + all depth dimensions at once.
- File asset created with unintended auto-name because naming gate was skipped.
- `POST`/`PUT`/`PATCH` tests fail due to unapproved or mismatched AI-generated payload.
- DB validation blocked by missing connection or uncertain correlation keys.
- Generated full-definition suites left unpruned despite subset user intent.
- Copy/move failures caused by stale ids or incorrect parent-id derivation.
- Incorrect validator type/chaining due to output inference from sibling tools.

## 9) Safety / Rollback
- Orchestration-first policy: avoid speculative writes before confirmation gate.
- Rollback delegated to downstream atomic skills used in execution slices.

## 10) Reuse Notes
- This is a composite orchestration card; it does not replace atomic endpoint cards.
- Preferred location for this class of cards: `docs/skills/composite-orchestration/`.
- Keep branch logic service-agnostic; keep target quirks in transient run artifacts.

## 11) Prompt Sequence Template (Default)
Use this condensed sequence in order, asking one question at a time and waiting for each answer before continuing:
1. "Do you want tests for all operations or a subset?"
2. "Should I generate only happy-path tests, or include negative tests too?"
3. "For negatives, limit to spec-defined non-2xx cases, or also propose AI-generated error cases for review?"
4. "What should I name the `.tst` file?"
5. "Some operations have additional request parameters (query params, request body fields). Want me to propose values from the spec for you to review, or would you prefer to provide them yourself?"
6. "For write-method payloads (`POST`/`PUT`/`PATCH`), do you want to provide payloads, review AI-proposed payloads, or explicitly allow autonomous fill?"
7. "Do you want validator/assertor tooling added automatically with a review pass?"
8. "Do you want DB-backed validation included where feasible?"
9. "Proceed with this plan summary: <scope + counts + tool chain + tst name + payload mode>?"

### 11.2 Subset Selection Prompt Pattern (Conversational)
When subset scope is requested:
1. Print operation catalog in human-readable format (method + path + optional summary).
2. Ask one conversational prompt: "Which endpoints do you want included? You can answer naturally (for example, 'all /customers endpoints' or 'login, transfer, withdraw')."
3. Echo resolved subset list and proceed after confirmation.

### 11.1) Agentic Context Adaptation
In non-conversational or agentic execution contexts (for example terminal-based agents, CI pipelines, or environments where blocking questions may be misinterpreted):
- Keep intake solicitation sequential (one unresolved question per prompt) to preserve clarity and traceability.
- When user intent is clear enough to satisfy the confidence gate (for example user provided service definition, scope, and test depth in one request), skip already-resolved questions and continue with the remaining unresolved question sequence.
- Never bypass mandatory gates:
  - `.tst` naming resolution,
  - explicit payload approval for happy-path `POST`/`PUT`/`PATCH` bodies.
- Reasonable defaults may be used only after the user explicitly opts in to those defaults.

## 12) Current Validation Status (2026-03-04)
- Initial draft defined.
- First live role-play execution completed (customer-subset scenario) and exposed hardening needs now codified in Section 6.7.
- Next validation target: rerun with strict subset prune + deterministic naming + response-output chaining + JSON validator/assertor defaults.
