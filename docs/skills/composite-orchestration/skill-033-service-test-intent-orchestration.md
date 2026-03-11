# Skill 033: Service-Test Intent Orchestration (Composite)

## 1) Skill Name
Gather user testing intent and orchestrate service-test authoring across atomic skills.

## 2) Objective
Convert underspecified requests (for example "create tests for this service") into a confirmed, executable broad authoring plan before any write operations are performed, and act as the canonical ambiguity-resolution entry point for service-test authoring when routing-critical intent is still unresolved.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting scope and preference input.

## 3) Scope
- In scope:
  - intent intake for underspecified endpoint-only or service-definition-driven requests
  - classify routing-critical ambiguity such as:
    - new `.tst` vs existing `.tst` vs one-client-only intent
    - broad generated coverage vs one operation/client
    - contract as generation source vs contract as planning input
    - REST vs SOAP branch
  - confidence-gated required-input resolution
  - operation/contract summarization for user scope decisions
  - progressive question bundles (coverage, negative tests, follow-on validation-enrichment intent) when broad authoring scope is still in play
  - high-level phase sequencing across broad generation/authoring, readiness stabilization, negative derivation, and optional validation follow-on
  - deterministic orchestration handoff to branch-specific orchestration cards or atomic/workflow skills once the branch is clear
  - staged execution only for branches that still require multi-step orchestration after clarification
- Out of scope:
  - replacing endpoint-specific behavior in atomic cards
  - acting as the mandatory route for already-specific direct requests that the routing registry can send to a smaller matching card
  - repeated clarified request-readiness remediation semantics now owned by Skill 058
  - repeated clarified validation-enrichment semantics now owned by Skill 057
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
  - follow-on validation-enrichment preferences
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
2. Determine whether any routing-critical dimensions are still unresolved:
   - target artifact scope (`new .tst`, `existing .tst`, or `single client/test`),
   - operation scope (`broad coverage` vs `single operation/client`),
   - source role (`generation source` vs `planning input`),
   - protocol branch (`REST` vs `SOAP`),
   - target asset/parent scope.
3. If all routing-critical dimensions are already resolved, hand off immediately using the routing registry instead of continuing broad intake:
   - direct single-client intent -> Skill 056,
   - direct validation-enrichment intent -> Skill 057,
   - direct file-level generation intent -> Skills 022-025,
   - direct existing-asset WSDL suite-generation intent -> Skill 055.
4. Resolve remaining missing required inputs by precedence:
   - explicit values in current prompt,
   - values confirmed earlier in current session,
   - relevant environment variables.

### 6.1.1 API Capability Preflight (Required)
2.1 Before any write operation, run branch-aware capability preflight (Skill 050):
  - apply the operation-class profile from Skill 050 Section 6.1,
  - classify outcomes and `404` disambiguation per Skill 050 Section 6.2,
  - record/reuse capability outcomes per Skill 050 Section 6.3.
2.2 Do not continue with optimistic endpoint assumptions when preflight has identified a fallback path.

### 6.1.2 Target Resolution Endpoint Selection (Required)
2.3 Resolve runtime targets with deterministic endpoint usage:
  - use `GET /v6/descendants/files` for filesystem-level discovery (`.tst`/`.pva` lookup, folder traversal),
  - use `GET /v6/children?id=<tst-id>` for root suite/object seed under a specific file,
  - use `GET /v6/descendants/assets?id=<asset-id>` for in-file object graph discovery (suites/tools/datasources/etc.).
2.4 Do not call `GET /v6/descendants/assets` with directory ids (for example `/TestAssets`); resolve file/asset ids first.
2.5 Parse descendants responses from `children` arrays and validate expected type before branching.

### 6.1.3 Authoring Efficiency Boundary (Required)
2.6 Follow global exemplar-efficiency policy from `docs/workflow/agent-workflow.md` (Decision Rule):
  - do not perform workspace-wide same-type exemplar scans before create/update when selected skills provide sufficient canonical payload-shape guidance,
  - use user inputs + endpoint schema + selected skill guidance as the primary payload-authoring path.
2.7 Allowed targeted readback remains:
  - inspect only in-scope producer/test artifacts needed for the active branch (for example baseline media-type evidence, generated test context, or post-write verification),
  - do not broaden this into general \"find another tool of same type\" lookup behavior.

### 6.2 Source Strategy Branch
5. If service definition/schema is missing or ambiguous, ask targeted question and wait.
6. If user has service definition/schema, confirm format and location type (URL vs file path).
7. If user only has endpoint and no service definition:
   - offer two paths:
    - one-client authoring (Skill 056 -> Skill 020/059/034 path), or
     - pause and supply service definition for broader generation (Skills 022-025 path).

### 6.2.1 Source Normalization (Internal, Not User-Facing)
8. Normalize user-provided source to API-compatible location fields:
   - URL input -> `location.url`
   - local/relative file path -> convert to workspace-accessible location:
     - prefer uploading/resolving to workspace file and using `location.id`, or
     - when runtime supports direct file URI, convert to canonical file URI form and validate reachability.
9. Do not ask users for `location.id` terminology directly unless they explicitly request API-level detail.
10. For deterministic parsing and traceability, materialize a transient source snapshot under `work/` before generation or contract-informed planning:
   - fetch/download the service definition using a deterministic transport (for example `curl`/HTTP client),
   - avoid UI/browser save dialogs,
   - parse operation/response metadata from the transient copy used for planning.

### 6.3 Scope Discovery (Progressive Sequence)
8. Ask concise high-impact intake questions one at a time, waiting for the user's answer before asking the next:
   - Coverage: all operations vs subset.
   - Depth: happy-path only vs include negative tests.
   - Follow-on enrichment: leave generated tests at client-level baseline validation only vs plan an explicit validation-enrichment pass after generation.
9. Ask advanced follow-ups only for selected branches, one question at a time:
   - Negative scope: spec-defined non-2xx only vs additional AI-suggested error cases.
   - Validation-enrichment follow-on: happy-path-only enrichment vs explicit negative/DB enrichment later.

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
   - planned generation/authoring branch,
  - target `.tst` file name,
   - write-request gate summary (including payload source for `POST`/`PUT`/`PATCH`),
   - planned follow-on phases (readiness remediation if needed, negatives, validation enrichment if approved),
   - assumptions/unknowns requiring confirmation.
14. Require explicit user confirmation before writes.
15. If confirmation is partial, execute only confirmed slices.

### 6.5 Orchestration Mapping (Examples)
16. Map confirmed plan to downstream skills:
   - Branch-specific orchestration handoff:
     - single-client authoring (endpoint-only or contract-informed) -> Skill 056
     - request-readiness/configuration remediation after broad generation or on existing targets -> Skill 058
     - direct validation-enrichment or approved follow-on enrichment -> Skill 057
   - Service-definition generation:
     - OpenAPI new `.tst` -> Skill 022
     - WSDL new `.tst` -> Skill 023
     - RAML new `.tst` -> Skill 024
     - XSD new `.tst` -> Skill 025
     - WSDL into existing `.tst` -> Skill 055
   - Structural stabilization:
     - generated subset pruning -> Skill 047
   - Runtime verification/diagnostics:
     - execution triad -> Skills 012/014

### 6.6 Scale Control
17. For large services, default to phased rollout unless user requests full breadth:
   - phase by tag/group/domain,
   - phase by top-N operations,
   - phase by profile (happy-path first, then negatives, then DB).

### 6.6.1 Post-Generation Readiness Stabilization
After generation and subset pruning when applicable, determine whether the selected/generated happy-path targets are sufficiently configured to act as the stable baseline for later phases.

Rules:
- Keep generation-critical pre-write gates in this card:
  - `.tst` naming resolution,
  - explicit payload approval for happy-path `POST`/`PUT`/`PATCH` bodies,
  - intake-resolved concrete path/query values that are already known before generation.
- If generated or existing happy-path targets still need broader request/configuration remediation after those gates are applied:
  - do not keep that generalized remediation logic in Skill 033,
  - hand off to `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`,
  - resume broad authoring only after Skill 058 verifies readiness.
- Skill 058 owns:
  - readiness heuristics,
  - AI-proposes vs user-provides remediation modes,
  - grouped REST/SOAP/DB remediation slices,
  - verification execution for readiness.
- Use the finalized ready happy-path baseline before creating negative variants or handing off to validation enrichment.

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
    1) apply intake-resolved happy-path request data during broad authoring,
    2) if generated or existing happy-path targets still need broader readiness/configuration work, hand off to Skill 058 and resume only after readiness verification,
    3) clone/create negative variants from the finalized happy baseline,
    4) run negative calibration execution and finalize negative expected-code settings,
    5) hand off approved happy-path validation enrichment to Skill 057,
    6) hand off negative validation enrichment only when explicitly requested.
  - do not attach validation tools to happy tests before cloning negatives; this prevents copied validator/assertor/diff children from leaking into negatives by default before validation-enrichment policy is applied.
  - if validation tools already exist on a happy-path parent before cloning, cloned negatives must remove inherited validation children unless user explicitly requested negative validation tooling.
20.2 Approved-plan completion guard (required):
  - after plan confirmation, do not interrupt execution with repeated proceed/stop prompts for known runtime failures unless a true blocker is hit,
  - treat business-level test failures as diagnostics outputs, not orchestration stop conditions,
  - continue remaining approved stages (for example negatives, calibration, validation tooling) and report residual failures in a final summary,
  - prompt the user only when a dependency is blocked (for example cannot resolve ids, cannot chain tool due invalid parent type, missing required source/schema with no safe fallback).
21. Validation-enrichment handoff policy:
  - if the approved plan includes explicit validation enrichment beyond client-level expected response handling, hand off to Skill 057 after generation, subset pruning, readiness stabilization, and negative calibration reach a stable state.
  - Skill 057 owns live-response baseline collection, happy-path validation bundle proposal, schema-source confirmation, response-vs-database enrichment, negative validation defaults, and validation-tool attachment behavior.
  - if the user requested generated tests only and did not approve a follow-on validation-enrichment pass, stop after the generation/negative-calibration phase.
22. Pre-execution URL readiness check (REST Client flows):
  - before invoking execution skills, sample targeted REST Clients and confirm `resource.literalText.fixed` is non-empty,
  - if empty URL is detected, stop execution and repair configuration before run; use Skill 058 when the branch now requires generalized readiness remediation rather than one local fix.
23. Intake-parameter materialization for happy-path URLs:
  - when user provides concrete path/query values during intake (for example `customerId=12212`), apply them to happy-path REST Client resource templates before cloning negatives,
  - do not leave unresolved placeholders (for example `{customerId}`) in happy-path URLs when concrete intake values were provided,
  - derive negative variants from the concretized happy baseline so spec-default negatives retain realistic context.

## 7) Validation
- No write operation occurs before explicit plan confirmation.
- No write operation occurs before `.tst` naming is resolved (explicit name or explicit auto-name approval).
- For `POST`/`PUT`/`PATCH` happy-path tests, no write occurs before payload source and payload content are explicitly approved.
- Generalized post-generation readiness remediation is handed off to Skill 058 rather than duplicated locally in Skill 033.
- Every confirmed branch maps to explicit downstream skills.
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
- Generated or existing happy-path targets remain underconfigured but are not handed off to Skill 058 before later phases proceed.
- Generated full-definition suites left unpruned despite subset user intent.
- Copy/move failures caused by stale ids or incorrect parent-id derivation.
- Negative derivation or follow-on enrichment begins before the happy-path baseline has been stabilized.

## 9) Safety / Rollback
- Orchestration-first policy: avoid speculative writes before confirmation gate.
- Rollback delegated to downstream atomic skills used in execution slices.

## 10) Reuse Notes
- This is a composite orchestration card; it does not replace atomic endpoint cards.
- Use this card when service-test authoring intent is still materially underspecified.
- If the request is already specific enough to match a direct branch in `docs/skills/skill-index.md`, route to that smaller card instead of forcing Skill 033.
- Keep this card focused on ambiguity resolution, generation-critical pre-write gates, and high-level phase sequencing.
- Use Skill 058 for generalized post-generation or existing-target readiness/configuration remediation, and use Skill 057 for detailed validation-enrichment behavior after readiness is stable.
- Preferred location for this class of cards: `docs/skills/composite-orchestration/`.
- Keep branch logic service-agnostic; keep target quirks in transient run artifacts.

## 11) Prompt Sequence Template (Default)
Use this condensed sequence in order for prompts that still require broad authoring intake, asking one question at a time and waiting for each answer before continuing. If steps 1-2 resolve to one-client intent, hand off to Skill 056 instead of continuing this broader sequence.
1. "Do you want a new `.tst`, tests added into an existing `.tst`, or one REST/SOAP Client only?"
2. "Do you want broad generated coverage, or just one operation/client?"
3. "Do you want tests for all operations or a subset?"
4. "Should I generate only happy-path tests, or include negative tests too?"
5. "For negatives, limit to spec-defined non-2xx cases, or also propose AI-generated error cases for review?"
6. "What should I name the `.tst` file?"
7. "For write-method payloads (`POST`/`PUT`/`PATCH`), do you want to provide payloads, review AI-proposed payloads, or explicitly allow autonomous fill?"
8. "After generation, should I leave the tests with client-level baseline validation only, or plan a follow-on validation-enrichment pass?"
9. "If you want that follow-on pass, should it also include response-vs-DB checks where feasible?"
10. "Proceed with this plan summary: <scope + counts + branch + tst name + payload mode + follow-on phases>?"

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
