# Skill 058: Request-Readiness Remediation Orchestration (Composite)

## 1) Skill Name
Detect and repair underconfigured REST/SOAP client tests or DB Tools before validation or later orchestration proceeds.

## 2) Objective
Convert request-readiness remediation intent into an execution plan that detects underconfigured existing or newly generated client tests or DB Tools, distinguishes them from intentionally negative tests and valid empty DB results, remediates request/configuration gaps through the appropriate client/DB lifecycle skills, verifies that the targets now produce meaningful runtime evidence, and then returns either verified-ready slices or an aggregated unresolved-blocker set to the calling workflow or user.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting remediation scope and request-value decisions.

## 3) Scope
- In scope:
  - detect underconfigured REST Client tests, SOAP Client tests, and DB Tools
  - support REST-only, SOAP-only, DB-only, and mixed selections in the same `.tst` or suite
  - classify selected targets by protocol/tool family before remediation
  - distinguish underconfigured happy-path tests from intentionally negative tests and distinguish incomplete DB Tools from valid empty DB outputs
  - remediate request/runtime readiness for existing or newly generated targets using grouped tool-family-aware slices
  - preserve inherited autonomous posture from Skill 033 broad-authoring runs
  - support request-value modes:
    - AI proposes, user confirms
    - user provides all
    - bounded autonomous remediation when inherited from Skill 033
  - bounded autonomous remediation per target:
    - one initial synthesis attempt
    - at most one evidence-driven correction
    - no open-ended retry loops
  - verify that remediated targets now produce meaningful runtime evidence
  - aggregate unresolved blockers for caller-owned interruption when bounded autonomous remediation cannot complete a target slice
  - return control to the calling workflow after readiness verification succeeds
- Out of scope:
  - broad service-test generation or new high-level authoring intake
  - validation bundle design and validator/assertor/diff attachment
  - datasource-driven large-scale parameterization as a primary workflow
  - replacing class-specific mutation semantics already owned by Skills 020, 034, and 015
  - inventing SQL strategy or DB-side setup/remediation
  - rewriting intentionally negative or intentionally sparse tests as though they were incomplete
  - explicit experimental direct endpoint exploration now owned by Skills 065 and 066
  - owning the final user-facing interruption UX when caller context delegates that responsibility back to Skill 033

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- Additive:
  - `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`
  - `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`

## 4) Inputs
- Required:
  - user objective to remediate request readiness for existing or newly generated tests or DB Tools
  - target scope anchor:
    - `.tst` file, and/or
    - suite/test/tool selection, and/or
    - explicit test names, tool names, or ids
- Optional:
  - caller context (for example validation enrichment or broad authoring follow-on after generation/subset stabilization)
  - caller posture when inherited from an upstream orchestration card
  - service definition/source when available to inform request remediation
  - preferred remediation mode (`AI proposes`, `user provides`, `bounded autonomous`)
  - known constraints on acceptable request values
  - known datasource context or parameterization context
  - known DB connection/query constraints

## 5) Preconditions
- API reachable and authenticated for all downstream atomic skills.
- Target tests or DB Tools are resolvable.
- Selected targets are executable, or focused execution can be run to gather readiness evidence.
- The remediation scope is limited to client tests or DB Tools whose requests/configuration can be safely updated through downstream lifecycle skills.

## 6) Procedure
### 6.1 Intake Classification
1. Confirm the request is request-readiness remediation intent, for example:
   - help configure these existing tests,
   - these client tests still need request data,
   - these tests are failing because the requests look default,
   - this DB Tool still needs its connection/query configured,
   - make these targets ready before validation.
2. If the request is still broad service-test creation/generation intent, route back through Skill 033 instead of continuing here.
3. Resolve target scope by precedence:
   - explicit values in the current prompt,
   - active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
   - values confirmed earlier in the current session,
   - caller context from Skill 057 or later Skill 033 handoff.
4. Ask scope once when unresolved:
   - which tests or DB Tools should be remediated?
4a. Resolve effective remediation posture from caller context before reopening remediation-mode choices:
  - when called from autonomous Skill 033 broad authoring, inherit autonomous posture through this card,
  - when no caller posture is provided, default to direct interactive remediation behavior.
4b. In inherited autonomous posture:
  - do not reopen a remediation-plan approval gate before writes,
  - execute bounded remediation from the allowed evidence ladder,
  - return unresolved targets as one aggregated blocker set to the caller rather than interrupting inline per target.
4c. When caller context is Skill 033, Skill 033 remains the owner of the final user-facing interruption for unresolved blockers returned from this card.

### 6.1.1 API Capability Preflight (Required)
5. Before any write operation, run branch-aware capability preflight via `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.
6. Use the matched Skill 050 profile for the active branch, reuse/classify outcomes through Skill 050, and do not continue with optimistic endpoint assumptions when Skill 050 selects a fallback route.

### 6.1.2 Target Resolution Endpoint Selection (Required)
7. Resolve runtime targets using the API-first target-resolution policy from `docs/workflow/agent-workflow.md` plus the endpoint-selection mechanics in `docs/skills/platform/skill-001-shared-introspection.md`.
8. Keep this card focused on remediation-specific scope, evidence, and handoff rather than duplicating the full endpoint-selection matrix.

### 6.2 Runtime Evidence and Readiness Detection
9. Gather or confirm runtime evidence for the selected targets:
  - prefer the selected targets' own execution traffic/results context,
  - if evidence is absent or stale, run focused execution using Skill 012.
10. Classify selected targets by protocol/tool family:
  - REST Client slice
  - SOAP Client slice
  - DB Tool slice
  - unsupported / out-of-scope slice
11. Mixed REST/SOAP/DB selections are a normal orchestration case:
  - remediate in grouped tool-family-aware slices rather than forcing one protocol-agnostic branch.
12. Determine whether a selected target appears underconfigured using combined evidence from configuration shape, runtime symptoms, and branch context.
13. Require either:
  - at least two independent underconfiguration signals, or
  - one strong configuration signal plus one matching runtime symptom,
  before classifying a target as underconfigured.
14. High-confidence underconfiguration signals may include:
  - REST/SOAP configuration-shape signals:
    - empty or default-like request payloads on supposed happy-path write methods,
    - unresolved placeholders,
    - missing required request fields implied by known request shape.
  - DB Tool configuration-shape signals:
    - blank or scaffold-like JDBC/query configuration,
    - missing SQL query text,
    - unresolved SQL placeholders or obviously incomplete connection fields.
  - matching runtime symptoms:
    - responses, failures, or tool errors that strongly imply incomplete configuration rather than purposeful negative testing or valid empty business results.
15. Failure alone is not enough, and default-like scalar values count only when surrounding semantics show that meaningful user values are expected.
16. Explicitly exclude targets that are clearly intended negatives or otherwise not remediation candidates, including:
  - naming,
  - expected-response-code strategy,
  - deliberate request mutation,
  - branch context,
  - `GET`/`DELETE` requests where an empty body is normal,
  - connectivity/auth/server failures with otherwise well-populated requests,
  - empty-but-valid DB resultsets or "no rows found" outcomes.
17. If some selected targets are ready and others are underconfigured:
  - summarize the slices separately,
  - make it explicit that this card owns only the underconfigured slice,
  - let the user remediate only the underconfigured slice first or defer part of that slice,
  - leave ready-slice continuation to the calling workflow or the user rather than continuing validation/enrichment from inside this card.

### 6.3 Remediation Planning
18. For underconfigured targets, determine the remediation mode from effective posture plus any explicit user override:
  - direct interactive mode defaults to AI proposes, user confirms,
  - direct interactive mode may instead use user provides all values,
  - direct interactive mode may use bounded autonomous remediation only when the user explicitly approves it,
  - inherited autonomous Skill 033 mode uses bounded autonomous remediation by default.
19. Build a per-target remediation inventory for each underconfigured target:
  - target name/id and tool family,
  - evidence supporting the underconfigured classification,
  - request/configuration gaps that must be fixed before the target can be considered ready,
  - candidate values or value sources when available,
  - unresolved decisions that still require user input or blocker promotion.
20. Source candidate remediation values in this order:
  - explicit user-provided values,
  - active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
  - values already confirmed earlier in the current session,
  - OpenAPI/WSDL/XSD hints (for example examples, defaults, enums, types, and requiredness),
  - values inferred from other tests or datasources within the same `.tst` only, regardless of which nested suite they live under.
21. Treat same-`.tst` evidence as the outer inference boundary by default:
  - do not inspect other `.tst` files for candidate business data,
  - do not perform ad-hoc direct service probing or trial-and-error business-data discovery unless the user explicitly approves autonomous runtime probing for that slice.
22. Use project context, service-definition, request-shape, or known DB Tool configuration evidence to identify candidate remediation fields:
  - REST: path/query/body inputs, base URLs, and environment-specific values,
  - SOAP: endpoint, request envelope values, SOAPAction, and related exposed request/transport fields,
  - DB Tool: JDBC driver/url/username/password handling, SQL query text, query placeholders, and confirmed parameter values without inventing SQL strategy.
23. When AI proposes values in direct interactive mode:
  - cite the source when possible,
  - label contract-only or same-`.tst` inferences as proposals until the user approves them,
  - do not silently treat example/sample literals as approved user values.
24. If evidence is insufficient to produce trustworthy concrete values:
  - in direct interactive mode:
    - generate best-guess proposal values that satisfy the known type/shape/requiredness constraints,
    - ask the user to correct, replace, or approve them before any write handoff,
    - do not escalate into autonomous live-service probing by default.
  - in inherited autonomous mode:
    - synthesize the strongest best-guess values available from the same allowed evidence ladder,
    - use them as the bounded initial remediation attempt,
    - do not escalate into autonomous live-service probing by default.
25. If datasource context is relevant, use Skill 051 only as supporting discovery, not as a substitute for explicit remediation approval.
26. Build a human-readable remediation plan summary before any downstream mutation handoff:
  - underconfigured target slice in scope,
  - any ready slice or deferred slice kept outside this card,
  - per-target missing request/configuration fields,
  - proposed values or value sources,
  - unresolved decisions that still require user input,
  - downstream leaf-skill mapping for the actual mutations.
27. Confirmation rule by posture:
  - direct interactive mode requires explicit user confirmation before remediation writes or downstream mutation handoff,
  - inherited autonomous mode skips that pre-write confirmation and proceeds under the bounded remediation loop below.
28. Do not return control to the caller as though remediation is complete while required remediation decisions remain unresolved.

### 6.3.1 Bounded Autonomous Remediation Loop
29. In inherited autonomous posture, execute bounded remediation per underconfigured target:
  - synthesize one initial request/configuration attempt from the allowed evidence ladder,
  - apply the appropriate downstream mutation handoff,
  - run focused verification for that target/slice.
30. Allow at most one second attempt for a target, and only when the first verification produced new narrowing evidence that changes the likely fix.
31. If the first verification does not produce new narrowing evidence, do not spend the second attempt on another blind guess; promote the target directly to a blocker instead.
32. After one evidence-driven correction attempt, stop retrying that target in the current remediation pass even if it is still unresolved.
33. For each unresolved target promoted to blocker, record:
  - target identity,
  - why readiness remains unresolved,
  - the initial synthesized request/configuration attempt,
  - the evidence-driven correction attempt when one occurred,
  - the best next candidate value or the missing fact that would disambiguate the next move.
34. Aggregate unresolved targets across the underconfigured slice into one blocker set for the caller:
  - do not interrupt inline per target,
  - do not drip-feed one blocker at a time when batching is possible,
  - leave the final user-facing interruption to the caller when caller context requires it.

### 6.4 Protocol-Aware Remediation Handoff
35. For REST slices:
  - route unconstrained / None-mode REST Client request updates through `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.
  - route same-operation value edits for existing service-definition-backed constrained REST Clients through `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`, using the REST Client API path for constrained JSON request payloads and the YAML path for same-operation resource/config surfaces.
36. For SOAP slices:
  - route request updates through `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`.
37. For DB Tool slices:
  - route configuration updates through `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`.
38. Do not flatten mixed REST/SOAP/DB remediation into one protocol-agnostic mutation path.
39. Carry forward values by posture:
  - direct interactive mode carries forward only explicitly confirmed remediation values into the downstream leaf skills,
  - inherited autonomous mode may carry forward synthesized values within the bounded remediation loop,
  - best-guess values never authorize open-ended retry loops.

### 6.5 Verification and Return-to-Caller
40. After remediation, run focused verification execution for the remediated slice.
41. Confirm that remediated targets now produce meaningful runtime evidence rather than default/empty request failures or obviously incomplete DB Tool configuration.
42. If verification still indicates underconfiguration:
  - in direct interactive mode, summarize what remains unresolved and prompt for the next missing decisions instead of pretending readiness is achieved,
  - in inherited autonomous mode, follow the bounded remediation loop rules above and then either mark the target ready or promote it to the aggregated blocker set.
43. If the user defers remediation or required decisions remain unresolved:
  - report the slice as `deferred` or `still unresolved`, not `ready`,
  - do not return that slice to Skill 057 for validation-bundle design,
  - do not present pre-remediation outputs as sufficient evidence for later validation planning.
44. Return control based on caller context:
  - when called from Skill 057, return to validation enrichment only for slices that are now verified ready from post-remediation evidence,
  - when called from Skill 033, return to the broader authoring orchestration after either:
    - readiness verification succeeds for the slice, or
    - an aggregated unresolved-blocker set is prepared for Skill 033 to present,
  - when invoked directly, stop after reporting `ready`, `deferred`, or `still unresolved` status unless the user asks for the next phase.

## 7) Validation
- Mixed REST-only, SOAP-only, DB-only, and mixed selections are all handled as normal orchestration cases.
- Underconfigured tests are identified only from combined evidence, not from failure alone.
- Intentionally negative tests are not rewritten as though they were incomplete.
- Empty-but-valid DB resultsets are not rewritten as though they were incomplete.
- Direct interactive remediation does not write before an explicit remediation plan summary is shown and confirmed.
- Underconfigured targets are walked through per target before downstream mutation handoff.
- Candidate-value sourcing stays within user-confirmed context, contract hints, and same-`.tst` evidence unless the user explicitly approves autonomous runtime probing.
- Same-`.tst` inference may use other tests or datasources anywhere in the same file, but not other `.tst` files on the server.
- When trustworthy concrete values are unavailable:
  - direct interactive mode keeps best-guess proposals non-writable until the user approves them,
  - inherited autonomous mode may use one bounded best-guess synthesis attempt plus at most one evidence-driven correction.
- Inherited autonomous remediation never retries a target more than twice in one pass.
- The second inherited-autonomous attempt must be evidence-driven; no second blind guess is allowed.
- REST request remediation routes through Skill 020 semantics.
- Existing constrained REST Client remediation routes through Skill 059 when the selected operation remains unchanged; constrained JSON request payloads are modeled from the request schema and written through REST Client API `GET/PUT/GET`, while same-operation resource/config surfaces continue to use the YAML path documented there.
- SOAP request remediation routes through Skill 034 semantics.
- DB Tool remediation routes through Skill 015 semantics without inventing SQL strategy.
- Remediated targets are verified by focused execution before being declared request-ready.
- Unresolved inherited-autonomous targets are aggregated into one blocker set for the caller instead of interrupting inline per target.
- Only verified-ready slices return to Skill 057 for validation enrichment.

## 8) Failure Modes
- All failing tests are incorrectly treated as underconfigured.
- Intentionally negative tests are overwritten under the assumption that they are incomplete.
- Empty-but-valid DB resultsets are mistaken for incomplete DB Tool configuration.
- Mixed REST/SOAP/DB selections are flattened into one incompatible remediation path.
- Candidate values are sourced by probing the live service or mining other `.tst` files without explicit user approval.
- Best-guess proposal values are treated as implicitly approved final request data.
- Inherited autonomous posture from Skill 033 is dropped and replaced with a fresh remediation confirmation gate.
- A second autonomous remediation attempt is made without new narrowing evidence from the first attempt.
- Autonomous remediation loops beyond one synthesis attempt plus one evidence-driven correction for the same target.
- Request remediation is declared successful without verification execution.
- Example or default values are silently treated as user-approved final request data.
- DB Tool remediation invents SQL strategy instead of limiting itself to incomplete-configuration repair.
- The card returns control before per-target remediation decisions are resolved.
- The card interrupts per target instead of returning one aggregated blocker set when batching was possible.
- A deferred or unresolved slice is treated as validation-ready by the caller.

## 9) Safety / Rollback
- Orchestration-first policy:
  - avoid speculative writes before the confirmation gate in direct interactive mode,
  - avoid open-ended retry loops in inherited autonomous mode by enforcing the bounded remediation loop.
- Rollback is delegated to downstream client-lifecycle skills used in the approved remediation slices.

## 10) Reuse Notes
- This is a composite remediation card; it does not replace atomic REST/SOAP client lifecycle skills or the DB Tool lifecycle skill.
- Use this card when the user's primary problem is that existing/generated tests or DB Tools are not request-ready yet.
- Skill 057 may hand off here before validation enrichment proceeds.
- Skill 033 may hand off here during broad authoring after generation/subset stabilization when the happy-path baseline is not yet ready for standard negatives, optional security branching, or later enrichment.
- When called from autonomous Skill 033 broad authoring, this card inherits that autonomous posture, performs bounded remediation, and returns any unresolved targets as one aggregated blocker set for Skill 033 to present.
- This card owns remediation for the underconfigured slice only; any already-ready slice remains with the caller/user for later validation or sequencing choices.
- Preferred location for this class of cards: `docs/skills/composite-orchestration/`.
## 11) Prompt Sequence Template (Direct/Interactive Default)
Use this condensed sequence in order for direct interactive prompts that still require clarification, asking one question at a time and waiting for each answer before continuing.
1. "Which tests or DB Tools should I help configure?"
2. "Do you want me to propose the missing request/config values from the contract and same `.tst` context, or do you want to provide them yourself?"
3. "Should I remediate the underconfigured slice now, or defer part of it and leave any already-ready targets untouched for the next phase?"
4. "Proceed with this remediation plan summary: <selected targets + readiness findings + tool-family slices + proposed request/config updates>?"
When called from autonomous Skill 033, skip this interactive sequence and use the bounded remediation loop plus aggregated blocker return instead.
