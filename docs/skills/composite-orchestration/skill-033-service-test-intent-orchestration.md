# Skill 033: Service-Test Intent Orchestration (Composite)

## 1) Skill Name
Gather user testing intent and orchestrate service-test authoring across atomic skills.

## 2) Objective
Convert underspecified requests (for example "create tests for this service") into a posture-resolved, executable broad authoring plan before any write operations are performed, and act as the canonical ambiguity-resolution entry point for service-test authoring when routing-critical intent is still unresolved.

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
  - progressive question bundles for broad authoring scope, including:
    - all operations vs subset
    - standard negative coverage
    - security coverage
    - response-vs-database follow-on intent when relevant
  - autonomous/no-interrupt execution mode after explicit user opt-in
  - template-backed approval artifact generation before writes in interactive mode
  - high-level phase sequencing across broad generation/authoring, readiness stabilization, standard negative derivation, optional security branching, calibration, and validation enrichment
  - deterministic orchestration handoff to branch-specific orchestration cards or atomic/workflow skills once the branch is clear
  - staged execution only for branches that still require multi-step orchestration after clarification
  - concise completion artifact generation at the end of the approved workflow slice
- Out of scope:
  - replacing endpoint-specific behavior in atomic cards
  - acting as the mandatory route for already-specific direct requests that the routing registry can send to a smaller matching card
  - repeated clarified request-readiness remediation semantics now owned by Skill 058
  - repeated clarified validation-enrichment semantics now owned by Skill 057
  - domain-specific business test recipes as mandatory defaults

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/structure/skill-047-generated-subset-prune.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
  - `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/templates/skill-033-approval-artifact-template.md`
  - `docs/templates/skill-033-completion-artifact-template.md`

## 4) Inputs
- Required:
  - user objective to create service tests
  - at least one initial anchor:
    - service endpoint, or
    - service definition/schema location (user-provided URL or file-system path)
- Conditionally required before first write:
  - target `.tst` file name, or explicit approval to use deterministic auto-name, or autonomous-mode defaulting to deterministic auto-name
  - for write methods (`POST`/`PUT`/`PATCH`): either an interactive-mode request-body strategy with approved concrete payloads, or autonomous posture that authorizes autonomous payload fill for each happy-path operation that requires a body
- Optional:
  - preferred service-definition format (`openapi`, `wsdl`, `raml`, `xsd`)
  - preferred operation scope (all/subset)
  - explicit request for autonomous/no-interrupt execution
  - approval to reuse same-`.tst` request values when contract-compatible
  - response-vs-database comparison intent and known DB constraints

## 5) Preconditions
- API reachable and authenticated for all downstream atomic skills.
- Parent workspace location for new/updated assets is resolvable.
- User intent has reached minimum clarity threshold before write execution begins.

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
   - direct file-level generation intent with explicit **new `.tst`** request -> Skills 022-025,
   - direct existing-asset WSDL suite-generation intent -> Skill 055.
3a. Broad-authoring continuation rule (required):
   - If this run entered through Skill 033 because the user asked for broad service-test creation help, resolving the `new .tst` generation branch does not terminate Skill 033 ownership.
   - In that case, Skills 022-025 are used only to generate the initial happy-path host asset for the selected service-definition branch.
   - After generation, control returns to Skill 033 for readiness stabilization, standard-negative derivation, optional security branching, calibration, and validation enrichment unless the user explicitly narrowed scope to generation-only.
4. Resolve remaining missing required inputs by precedence:
   - explicit values in current prompt,
   - values confirmed earlier in current session,
   - relevant environment variables.
5. Detect whether the user explicitly requested autonomous/no-interrupt behavior.
   - if yes, use autonomous posture:
     - ask only blocker questions required to resolve minimally viable inputs,
     - apply the autonomous default-decision ledger,
     - skip the pre-write approval artifact interruption,
     - continue execution once blockers are cleared.
   - if posture is still unresolved, ask one concise posture question before broad intake continues.
   - preferred wording: "Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the plan together? Just say 'autonomous' or 'auto', or 'interactive'."

### 6.1.1 Direct-Generation Boundary (Required)
6. Treat help-style or outcome-level prompts (for example, `help me create tests for this OpenAPI`) as underspecified by default.
7. Consider direct file-level generation intent resolved only when the user explicitly requests creating a **new `.tst`** from a service definition.
8. If that explicit new-file intent is missing, remain in Skill 033 and ask a concise routing clarification (`new .tst`, `existing .tst`, or `one-client`).
   - preferred wording: "Should I create a new `.tst`, add tests into an existing `.tst`, or create one client/test only?"

### 6.1.2 API Capability Preflight (Required)
9. Before any write operation, run branch-aware capability preflight via `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.
10. Use the matched Skill 050 profile for the active branch, reuse/classify outcomes through Skill 050, and do not continue with optimistic endpoint assumptions when Skill 050 selects a fallback route.

### 6.1.3 Target Resolution Endpoint Selection (Required)
11. Resolve runtime targets using the API-first target-resolution policy from `docs/workflow/agent-workflow.md` plus the endpoint-selection mechanics in `docs/skills/platform/skill-001-shared-introspection.md`.
12. Follow Skill 001 for exact-name-first discovery, file-vs-asset endpoint choice, response-collection normalization, and post-mutation re-resolution.
13. Keep this orchestration layer focused on branch choice and handoff; do not restate the full endpoint-selection matrix here.

### 6.2 Source Strategy Branch
14. If service definition/schema is missing or ambiguous, ask one targeted question and wait.
15. If the user has a service definition/schema, confirm format and location type (URL vs file path).
   - when the blocker is that the user says they have a service definition but has not yet provided where it is, preferred wording is: "Please send me where to find your service definition, whether that's a URL or a local filesystem path."
16. If the user only has an endpoint and no service definition:
   - in autonomous posture, route to one-client authoring (Skill 056 -> Skill 020/059/034 path) because broad generated authoring lacks the minimally viable service-definition input,
   - in interactive posture, offer one-client authoring (Skill 056 -> Skill 020/059/034 path), or
   - pause and ask for a service definition when the user wants broader generation.
17. Normalize user-provided source to API-compatible location fields internally:
   - URL input -> `location.url`
   - local/relative file path -> convert to workspace-accessible location:
     - prefer uploading/resolving to workspace file and using `location.id`, or
     - when runtime supports direct file URI, convert to canonical file URI form and validate reachability.
18. For deterministic parsing and traceability, materialize a transient source snapshot under `work/` before generation or contract-informed planning.

### 6.3 Scope Discovery (Progressive Sequence)
19. Resolve intake in stage order:
   - posture and routing blockers first,
   - broad-operation scope second,
   - later plan choices only after the selected operation slice is known.
20. In interactive mode, after routing and source blockers are clear, ask only unresolved high-impact planning questions still needed before execution:
   - Coverage: all operations vs subset.
   - Standard negatives: include standard negative coverage or happy-path only.
   - Security: include security coverage or omit it.
   - `.tst` naming when the target host asset is still unresolved.
   - Payload strategy only for body-bearing happy-path operations in the selected scope.
21. In autonomous mode, use this default-decision ledger unless the user already constrained the run:
   - deterministic `.tst` auto-name when no host asset/name was explicitly provided,
   - full operation scope unless a subset was specified,
   - standard negative coverage enabled,
   - security coverage enabled,
   - validation enrichment enabled,
   - response-vs-database checks deferred from default intake,
   - autonomous payload fill enabled for body-bearing happy-path operations.
   - Autonomous mode is a true no-interrupt execution path: ask only blocker questions, do not pause for planning summaries or approval artifacts once blockers are cleared, and stop as `blocked` rather than degrading into extended interactive intake when a blocker remains unresolved.
22. When scope narrowing is needed, always print a human-readable operation catalog before asking the user to choose:
   - show method + path (+ short summary when available),
   - include a few recommended presets when helpful,
   - allow free-text selection,
   - support exact path, prefix group, comma lists, and keyword matches,
   - print the resolved operation list before execution,
   - if confidence is low, ask one clarifying question.
   - do not ask the all-versus-subset question first and present the operation list only afterward; present the list as part of the same user-facing question.

### 6.3.1 Mandatory Artifact and Write-Request Gates
23. Before any create/copy/write action, resolve target test-asset naming according to posture:
   - interactive mode: ask the user for the desired `.tst` file name, or ask for explicit approval to use deterministic auto-name,
   - autonomous mode: use deterministic auto-name unless the user already supplied a target host/name.
24. For write methods (`POST`/`PUT`/`PATCH`), resolve request payload strategy according to posture before creating tests:
   - interactive mode: user provides payload, AI proposes payload for approval/edit, or AI fills autonomously only when the user explicitly opts in,
   - autonomous mode: fully autonomous payload fill is the default unless the user already constrained payload handling.
25. Treat payload as unresolved until approval is explicit in interactive mode:
   - if payload is AI-proposed, print a compact preview and ask for approve/edit,
   - do not write REST Client body fields while payload approval is unresolved.
   - autonomous mode skips this payload-review interruption and proceeds once minimally viable inputs are resolved.
26. For happy-path path/query/body values that are not yet concrete, classify each value as one of:
   - already confirmed,
   - contract-derived proposal,
   - same-`.tst` reuse proposal,
   - requires Skill 058 remediation.
27. Value-handling rule by posture:
   - interactive mode: do not silently invent concrete values after approval; unresolved values must stay visible in the approval artifact.
   - autonomous mode: values proposed or synthesized from contract/session/same-`.tst` evidence may proceed without a pre-write review step, but surprising gaps that cannot be resolved from those allowed sources remain blockers.

### 6.4 Approval Artifact + Confirmation (Interactive Required)
28. In interactive mode, build the pre-write approval artifact using `docs/templates/skill-033-approval-artifact-template.md`.
29. The approval artifact must contain only the compact preamble plus Sections `A-E` defined by the template.
30. Populate the compact preamble with:
   - `Defaults applied: ...` or `none`
31. Populate Section `A) Scope and Target` with:
   - target `.tst`
   - selected operations in stable order.
32. Populate Section `B) Happy-Path Configuration Blueprint` with one entry per selected happy-path test in stable operation order.
33. Populate Section `C) Negative and Security Coverage Plan` with one entry per selected operation:
   - list named standard-negative families,
   - render `Security Branch Planned: yes|no`,
   - do not show target counts,
   - do not expose internal structure-detection logic or planner rationale unless it directly explains a blocked or review item.
34. Populate Section `D) Review Items / Remediation Hotspots` only with issues that genuinely require user direction or approval.
   - use `R<n>` ids in stable order,
   - if no review items remain, emit `none`.
35. Populate Section `E) Approval Question` with the explicit proceed question for the exact planned branch.
36. If review items exist, instruct the user to reply by review id and do not start writes until those items are resolved and the user explicitly approves proceeding.
37. If no review items exist, a simple explicit approval is sufficient.
38. If confirmation is partial, execute only the confirmed slices.
   - autonomous mode skips this entire approval-artifact interruption and proceeds directly into execution once blocker questions are resolved.

### 6.5 Orchestration Mapping
39. Map the confirmed plan to downstream skills:
   - single-client authoring -> Skill 056
   - request-readiness/configuration remediation after broad generation or on existing targets -> Skill 058
   - validation enrichment after a stable baseline exists -> Skill 057
   - OpenAPI new `.tst` -> Skill 022 as the happy-path generation phase; return to Skill 033 afterward unless the user explicitly narrowed scope to generation-only
   - WSDL new `.tst` -> Skill 023 as the happy-path generation phase; return to Skill 033 afterward unless the user explicitly narrowed scope to generation-only
   - RAML new `.tst` -> Skill 024 as the happy-path generation phase; return to Skill 033 afterward unless the user explicitly narrowed scope to generation-only
   - XSD new `.tst` -> Skill 025 as the happy-path generation phase; return to Skill 033 afterward unless the user explicitly narrowed scope to generation-only
   - WSDL into existing `.tst` -> Skill 055
   - generated subset pruning -> Skill 047
   - execution/results/traffic verification -> Skills 012 and 014
   - penetration-testing security branch -> Skill 062

### 6.6 Scale Control
40. For large services, default to phased rollout unless the user requests full breadth:
   - phase by tag/group/domain,
   - phase by top-N operations,
   - phase by profile (happy-path first, then negatives, then DB).

### 6.7 Post-Generation Readiness Stabilization
41. After generation and subset pruning when applicable, determine whether the selected/generated happy-path targets are sufficiently configured to act as the stable baseline for later phases.
42. Keep generation-critical pre-write gates in this card:
   - `.tst` naming resolution,
   - explicit payload approval for happy-path `POST`/`PUT`/`PATCH` bodies,
   - intake-resolved concrete path/query values that are already known before generation.
43. If generated or existing happy-path targets still need broader request/configuration remediation after those gates are applied:
   - do not keep that generalized remediation logic in Skill 033,
   - hand off to Skill 058,
   - resume broad authoring only after Skill 058 verifies readiness.
44. Use the finalized ready happy-path baseline before creating standard negative variants, creating security branches, or handing off to validation enrichment.
44a. Treat that verified-ready happy-path baseline as the execution authority for any later copied security branch:
  - do not require a separate interim runtime re-validation of the copied security branch just to reconfirm the copied request,
  - reserve security-branch execution for explicit security-specific follow-up or an explicit user-approved end-of-workflow full `.tst` run.
45. When happy-path and standard-negative slices are both approved and ready after calibration, prefer one combined validation-enrichment handoff to Skill 057 rather than serializing them into separate user-visible validation phases.
46. Split validation enrichment into separate follow-up phases only when a real readiness split, blocker, or explicit user instruction requires it.

### 6.8 Structural Resolution and Suite Placement
47. Detect structure per selected operation, not only at the whole-file level:
   - operation-level layout when the operation has a dedicated suite containing its happy-path test/client,
   - spec-level flattened layout when operation tests/clients are direct children of a common spec suite.
48. If the file is mixed, resolve layout per operation rather than forcing one global classification.
49. Structural placement rules:
   - operation-level layout -> create nested `Negative Test Suite` and `Security Test Suite` under that operation suite,
   - spec-level flattened layout -> create sibling `Negative Test Suite for <Operation>` and `Security Test Suite for <Operation>` under the shared spec-level parent, then reorder those siblings so they follow the matching happy-path operation.
50. Only block an operation slice when its parent context cannot be resolved deterministically after current-read evidence.
51. Continue unaffected slices when safe.

### 6.9 Execution Hardening Rules (Required)
52. Enforce preflight route selection for write-phase calls.
53. Enforce ambiguous-write reconciliation before retry:
   - for any write with ambiguous command output, run deterministic readback reconciliation before deciding retry,
   - treat confirmed post-write state as success and continue orchestration,
   - for create branches, if the same-name `.tst` is found after ambiguous output, do not blindly re-post create.
54. Enforce subset pruning immediately after generation when subset scope was approved.
55. Use deterministic naming with non-cumulative labels:
   - happy path: `<operation> [happy]`
   - standard negatives: `<operation> [neg-<family-key>]`
   - copied security client: `<operation> [security]`
56. Negative request differentiation guard:
   - each standard negative must differ from the happy baseline at request level for the intended failure mode,
   - if a proposed mutation is a no-op, stop and re-derive the family or omit it.
57. Standard-negative planning algorithm:
   1. derive candidate standard-negative failure-mode families from the contract and request shape in this order:
      - explicit non-2xx response families documented in the service definition,
      - missing required input,
      - null or empty-value violations when those are materially different from omission,
      - enum/type/format/length/range/cardinality violation,
      - unsupported or malformed body representation when the request shape makes that mutation meaningful,
      - protocol-level negatives such as invalid HTTP method, unsupported media type, or unacceptable `Accept` negotiation when the contract/runtime context makes them plausible,
      - unexpected extra-field/body-shape violations when the contract indicates closed-world request structure,
      - duplicate/conflict state violations when the operation semantics imply uniqueness or conflict handling,
      - missing resource when a syntactically valid absent identifier is plausible;
   2. require a plausible request mutation for each family;
   2a. treat the candidate-family list as heuristic input to dynamic reasoning, not as a mandatory checklist that must be exhausted on every operation;
   3. deduplicate by intended failure mode;
   4. apply a first-pass cap of five families per operation;
   5. if more than five high-confidence families exist, keep the strongest five in scope;
   6. if only weak evidence exists, fall back to the smaller justified set;
   7. five is a ceiling, not a target.
58. Security branching policy:
   - in interactive mode, security is a separate intake decision from standard negatives,
   - in autonomous mode, security is enabled by default unless the user constrained the run otherwise,
   - security branches are created by copying the finalized happy-path client into the security suite and attaching a Penetration Testing Tool under the REST Client `Traffic Object` via Skill 062,
   - do not create ad hoc security REST mutations as a substitute for the Penetration Testing Tool branch.
58a. Interim execution-scope rule for security branches:
   - when Skill 033 performs focused execution for readiness verification, calibration, traffic inspection, diagnostics, or validation-design evidence, exclude security suites/tests from that execution scope,
   - use the already verified-ready happy-path baseline plus the standard-negative slice as the internal runtime-evidence set,
   - do not pull copied security branches into those interim runs merely because they exist in the same `.tst`,
   - if the user later wants an end-to-end/full-`.tst` execution, security suites may be included at that explicit follow-up stage.
59. Use fresh-id guard after any rename/copy/create operation.
60. Use parent-resolution guard for copy/move operations.
61. Negative expected-code calibration gate:
   - after creating standard negative variants, run calibration execution before finalizing expected codes,
   - update expected codes to match actual service behavior when the contract does not make the response explicit,
   - log surprising response codes as service-behavior observations.
62. Use read-merge-write PUT safety for any pre-configured tool update.
63. Validation sequencing guard:
   1. apply approved happy-path request data during broad authoring,
   2. hand off broader readiness/configuration work to Skill 058 when needed,
   3. clone/create standard negative variants from the finalized happy baseline,
   4. create the security branch when enabled,
   5. run happy-path/standard-negative interim execution only on the non-security slices needed for calibration or runtime evidence,
   6. run standard-negative calibration and finalize expected-code settings,
   7. hand off happy-path and standard-negative validation enrichment to Skill 057 as one combined pass when both slices are approved and ready,
   8. do not attach validator/assertor/diff tools to security branches,
   9. treat any later full-`.tst` execution that includes security suites as an explicit end-of-workflow follow-up rather than part of the default internal execution path.
64. Approved-plan completion guard:
   - after plan confirmation, do not interrupt execution with repeated proceed/stop prompts for known runtime failures unless a true blocker is hit,
   - continue remaining approved stages where safe,
   - prompt the user only when a true dependency is blocked.

### 6.10 Completion Artifact
65. After the approved workflow slice finishes, build the completion artifact using `docs/templates/skill-033-completion-artifact-template.md`.
66. The completion artifact must remain concise by default and contain only Sections `A-C` from the template.
67. For straightforward successful runs, keep the artifact to a compact scan:
   - outcome summary,
   - work performed,
   - remaining/follow-up items.
68. For partial or blocked runs, keep the same structure but let Section `C` carry the unresolved or deferred items.
68a. Do not emit a successful completion artifact immediately after happy-path generation when autonomous defaults or the approved broad-authoring plan still imply downstream negative, security, readiness, or validation work.
68b. If downstream phases cannot proceed after generation, report the run as `partial` or `blocked` and name the dependency or blocker rather than silently ending at generation.
69. After reporting completion, optional follow-up offers may include:
   - response-vs-database comparison when it was not already in scope,
   - an end-to-end/full-`.tst` execution, including security suites when the user wants that broader final run.

## 7) Validation
- Interactive mode does not start writes before explicit approval of the template-backed approval artifact.
- Autonomous mode skips the approval artifact and proceeds directly once blocker questions are resolved.
- No write operation occurs before `.tst` naming is resolved.
- Interactive `POST`/`PUT`/`PATCH` happy-path tests do not write before payload source and payload content are explicitly approved.
- Autonomous `POST`/`PUT`/`PATCH` happy-path tests default to autonomous payload fill unless the user constrained payload handling differently.
- Generalized post-generation readiness remediation is handed off to Skill 058 rather than duplicated locally.
- Standard-negative coverage is dynamic and may legitimately produce fewer than five evidence-backed families.
- Interactive intake treats standard negatives and security coverage as separate decisions.
- Autonomous mode defaults both standard negatives and security coverage on unless the user constrained the run otherwise.
- Response-vs-database comparison is not part of default intake; unless the user explicitly requested it earlier, it is offered only as an optional post-completion follow-up.
- Approval artifacts show named standard-negative families rather than target counts.
- Approval review ids appear only for real user-decision points.
- Completion artifacts remain concise and do not re-explain internal orchestration logic.
- A Skill 033 autonomous broad-authoring run is not complete merely because its direct generation leaf succeeded; completion requires either downstream phase execution or an explicit `partial`/`blocked` outcome when later phases cannot proceed.
- Runtime verification uses execution-triad skills for executed slices.
- Internal execution used for readiness evidence, calibration, traffic inspection, diagnostics, or validation design excludes copied security suites by default.
- Security-suite execution is reserved for explicit security-specific follow-up or an explicit user-approved end-of-workflow full `.tst` run.

## 8) Failure Modes
- Overlong questionnaire causes user drop-off.
- Ambiguous source format/location leads to incorrect generation branch.
- Scope explosion when user requests all operations plus multiple follow-on branches at once.
- Autonomous mode still pauses for planning summaries/approval artifacts and no longer feels autonomous.
- Autonomous broad-authoring continues without minimally viable source input instead of rerouting to one-client work or stopping blocked.
- File asset created with unintended auto-name because naming gate was skipped.
- `POST`/`PUT`/`PATCH` tests fail due to unapproved or mismatched AI-generated payload.
- Autonomous one-client or broad-authoring flow stalls on a host-choice question even though deterministic new-host defaulting should have been applied.
- Generated or existing happy-path targets remain underconfigured but are not handed off to Skill 058 before later phases proceed.
- Generated full-definition suites are left unpruned despite subset intent.
- Security coverage is silently bundled into standard negatives during interactive intake.
- Focused/interim execution pulls copied security suites into calibration, traffic, or validation-design runs and causes avoidable delay.
- Response-vs-database comparison re-enters the default intake flow instead of being left for explicit earlier request or post-completion follow-up.
- Approval artifact exposes internal planner reasoning instead of only user-relevant decisions.
- Completion artifact becomes so verbose that it is no longer scannable.

## 9) Safety / Rollback
- Orchestration-first policy: avoid speculative writes before the required interactive approval gate or before autonomous blocker inputs are resolved.
- Rollback remains delegated to the downstream atomic/composite skills used in the approved execution slices.

## 10) Reuse Notes
- This is a composite orchestration card; it does not replace atomic endpoint cards.
- Use this card when service-test authoring intent is still materially underspecified.
- If the request is already specific enough to match a direct branch in `docs/skills/skill-index.md`, route to that smaller card instead of forcing Skill 033.
- Keep this card focused on ambiguity resolution, template-backed approval/completion artifacts, generation-critical pre-write gates, and high-level phase sequencing.
- Use Skill 058 for generalized readiness/configuration remediation and Skill 057 for detailed validation-enrichment behavior after readiness is stable.
- Preferred location for this class of cards: `docs/skills/composite-orchestration/`.

## 11) Prompt Sequence Templates
Use these condensed sequences to preserve posture-aware intake. If the top-level route resolves to one-client intent, hand off to Skill 056 instead of continuing broad Skill 033 intake.
### 11.1) Interactive Path
1. If posture is unresolved: "Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the plan together? Just say 'autonomous' or 'auto', or 'interactive'."
2. "Should I create a new `.tst`, add tests into an existing `.tst`, or create one client/test only?"
3. If broad authoring continues: present the operation list first, then ask: "Here are the operations I found. Do you want all of them or a subset?"
4. "Do you want me to also include negative tests?"
5. "Do you want me to add security tests?"
6. "What should I name the `.tst` file?" (omit when target host/name is already resolved)
7. "For requests that include a body, do you want me to propose payloads for your approval, propose payloads without approval, or would you like to provide payloads yourself? You can say: approval, no approval, or manual."
8. "Reply to any review items by id if they remain, then approve the artifact if you want me to proceed."
### 11.2) Autonomous Path
1. If posture is unresolved: "Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the plan together? Just say 'autonomous' or 'auto', or 'interactive'."
2. "Should I create a new `.tst`, add tests into an existing `.tst`, or create one client/test only?"
3. If broad authoring continues and the service definition is usable, resolve full scope by default unless the user already constrained the operation slice.
4. If broad authoring was requested but only an endpoint is available, reroute to Skill 056 for one-client authoring rather than forcing broad generation without minimally viable input.
5. If a host asset is still unresolved for autonomous work, default to creating a deterministic new host asset rather than interrupting solely for host-choice approval.
6. Once blocker inputs are resolved, continue execution without a pre-write approval artifact.

### 11.3) Agentic Context Adaptation
In non-conversational or agentic execution contexts:
- skip questions the user has already resolved,
- preserve the same posture rules:
  - interactive mode keeps explicit approval of the approval artifact before writes,
  - autonomous mode asks only blocker questions and otherwise continues execution,
- never bypass mandatory gates:
  - minimally viable routing/source input,
  - `.tst` naming resolution through either explicit choice or authorized defaulting,
  - payload handling through either interactive approval or autonomous posture,
