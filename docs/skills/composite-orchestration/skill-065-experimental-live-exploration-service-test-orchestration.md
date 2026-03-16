# Skill 065: Experimental Live-Exploration Service-Test Orchestration (Composite)

## 1) Skill Name
Explicit opt-in experimental broad service-test authoring through live API exploration before SOAtest authoring.

## 2) Objective
Provide an additive experimental broad-authoring lane for REST/OpenAPI service testing that replaces the stable generation-first discovery loop with a cluster-aware exploration-first sequence: analyze the selected service definition, perform bounded direct endpoint exploration, capture transient stateful exploration evidence, normalize the generated template suite around a local `BASEURL` variable, author SOAtest assets from that evidence into a canonical cluster-local `.tst` structure, hand a validation-ready per-slice packet into Skill 067, and run only the minimum focused SOAtest verification needed to confirm persisted behavior and approved chained tooling.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting scope, opt-in, and execution posture.

## 3) Scope
- In scope:
  - explicit opt-in experimental broad authoring for REST/OpenAPI service definitions
  - posture-first intake, operation-catalog-based scope selection, negative/security and optional DB follow-on decisions, deterministic host-asset naming, and exploration-backed request-value/payload gates owned directly by this card
  - delegated-branch posture inheritance, batched blocker return, partial-approval handling, and no-early-completion ownership for the experimental lane
  - exploration-first orchestration before SOAtest writes
  - delegation to Skill 066 for cluster-aware direct API exploration rules and evidence legitimacy
  - canonical authored structure with a disabled generated template suite plus sibling `AI Generated Tests`, including post-generation `BASEURL` environment insertion and generated-template REST Client host normalization to `${BASEURL}` before later copies inherit that form
  - reference-only lookup of other `.tst` assets when useful for orientation or comparison, while keeping the current host asset's disabled generated template suite as the only normal copy source for authored coverage
  - cluster-local `Happy Path`, narrow `Optional / Deferred`, `Negative Coverage`, and `Security Coverage`
  - stateful happy-path authoring with baseline snapshots, safe reference reads, lifecycle flows, copyable scenario sub-suites, and final cleanup of empty happy-path child suites that ended with no viable authored content
  - evidence-backed standard-negative derivation with required per-slice negative-accounting outcomes plus required per-operation security-accounting, lifecycle-preferred security target selection, and security-branch sequencing
  - required happy-path validation-enrichment follow-through with a caller-owned Skill 65 -> Skill 67 handoff packet that treats the Skill 066 ledger as advisory rather than as a rigid validation contract
  - focused verification through Skill 074 disable-run-restore sandboxing
  - bounded post-authoring correction when focused SOAtest verification reveals persisted authoring/configuration mismatch
  - concise completion reporting for the experimental lane
- Out of scope:
  - stable default routing for broad authoring when experimental opt-in is absent
  - SOAP or non-OpenAPI broad authoring
  - replacing atomic mutation/execution owners
  - copying authored suites, tests, tools, or validation branches from other `.tst` assets into the new host `.tst`
  - weakening stable Skills 033, 057, or 058 in place
  - turning direct endpoint exploration into a general-purpose baseline outside this explicit lane

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
  - `docs/skills/structure/skill-074-set-disabled-state.md`
- Additive:
  - `docs/skills/platform/skill-022-tst-create-from-openapi.md`
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
  - `docs/skills/structure/skill-047-generated-subset-prune.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
  - `docs/skills/structure/skill-070-copy-tool.md`
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - `docs/skills/validation/skill-016-xml-assertor-workflow.md`
  - `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/templates/skill-065-approval-artifact-template.md`
  - `docs/templates/skill-065-completion-artifact-template.md`

## 4) Inputs
- Required:
  - user objective to create service tests through the experimental exploration-first lane
  - explicit experimental opt-in signal
  - OpenAPI location or already-resolved OpenAPI-backed service-definition reference
- Conditionally required before writes:
  - target `.tst` name or explicit approval to use deterministic auto-name in interactive mode
  - minimally viable auth/base URL posture for direct exploration and downstream REST Client configuration, including credentials from explicit user input or an approved secret reference, or enough context to classify `blocked-auth`
- Optional:
  - subset selection
  - autonomous/no-interrupt execution posture
  - standard-negative coverage intent
  - security coverage intent
  - optional DB comparison intent
  - request-value/payload handling preference for body-bearing happy-path operations in interactive mode
  - user-supplied request values or constraints

## 5) Preconditions
- API reachable and authenticated for the downstream API branches used in this lane.
- The selected source is REST/OpenAPI, not SOAP/WSDL or another generation family.
- The user has explicitly opted into the experimental lane.
- The target workspace location for the authored `.tst` is resolvable.

## 6) Procedure
### 6.1 Intake and Explicit Opt-In Gate
1. Confirm the request is broad service-test authoring intent rather than a narrow direct lifecycle request.
2. Confirm the user explicitly wants the experimental exploration-first branch, for example:
   - `use the experimental branch`
   - `try the exploration-first approach`
   - `use live endpoint exploration before authoring`
3. If experimental opt-in is absent, route to stable Skill 033 instead of continuing here.
4. If the source is not REST/OpenAPI broad authoring, stop the experimental branch and route to stable Skill 033.
5. Resolve missing required inputs by this precedence before asking the user to restate them:
   - explicit values in the current prompt
   - active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`)
   - values confirmed earlier in the current session
   - relevant environment variables already configured for this target
6. If execution posture is still unresolved, ask exactly:
   - `Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the experimental plan together? Just say 'autonomous' or 'auto', or 'interactive'.`

### 6.1.1 Experimental Intake Contract
7. In interactive mode, ask only unresolved high-impact intake questions and keep them in this order:
   - operation catalog + all operations vs subset
   - standard negative coverage yes/no
   - security coverage yes/no
   - target `.tst` name or approval to use deterministic auto-name
   - exploration-backed request-value/payload handling for body-bearing happy-path operations (`approval`, `no approval`, or `manual`)
8. When scope narrowing is needed, always present the operation catalog before asking all-vs-subset or subset-selection questions:
   - show method + path + short summary when available
   - include a few recommended presets when helpful
   - allow free-text selection
   - support exact path, prefix group, comma lists, and keyword matches
   - echo the resolved operation list before exploration proceeds
   - if confidence is low, ask one clarification question
9. In autonomous mode, use this default-decision ledger unless the user already constrained the run:
   - deterministic `.tst` auto-name when no target host/name was explicitly provided
   - active-project-local asset placement defaults when the user did not choose another destination
   - full operation scope unless a subset was specified
   - standard negative coverage enabled
   - security coverage enabled
   - exploration-backed validation enabled
   - response-vs-database comparison deferred from default intake
   - exploration-backed request bundles accepted for body-bearing happy-path operations once minimally viable auth/business inputs are resolved
10. Autonomous mode is a true no-interrupt execution posture:
   - ask only blocker questions
   - do not surface generic planning summaries or approval artifacts before the write phase
   - once blockers are cleared, continue execution until a real blocker or completion state is reached

### 6.1.2 Naming and Exploration-Backed Request-Value Gate
11. Before any create/copy/write action, resolve target `.tst` naming by posture:
   - interactive mode: ask for the desired `.tst` name or for explicit approval to use deterministic auto-name
   - autonomous mode: use deterministic auto-name unless the user already supplied a target host/name
12. For body-bearing happy-path operations, resolve request-value/payload handling by posture before writing REST Client configuration:
   - interactive mode: the user may provide values directly, approve the exploration-backed request bundle, or authorize no-approval application of that bundle
   - autonomous mode: exploration-backed request bundles may proceed without pre-write review once minimally viable auth, business data, and request-shape inputs are resolved
13. In interactive mode, do not write body/path/query values derived from exploration while request/value approval is unresolved:
   - keep unresolved values visible in the approval artifact and/or review items
   - do not silently convert unresolved exploration proposals into writes
14. In autonomous mode, surprising gaps that cannot be resolved from explicit input, project context, same-run evidence, or the exploration bundle remain blockers rather than reasons to drift back into extended interactive intake

### 6.1.3 API Capability Preflight (Required)
15. Before any write or execution operation, run branch-aware capability preflight via `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.
16. Use the matched Skill 050 profile for the active branch and do not continue with optimistic endpoint assumptions when Skill 050 selects a fallback route.

### 6.1.4 Phase Integrity and Autonomous Execution Discipline
16a. Apply the global mandatory-phase-integrity rule from `docs/workflow/agent-workflow.md`. In this card, the in-scope required phases are happy-path authoring, negative accounting/authoring, security accounting/authoring, validation enrichment, and verification unless the user explicitly narrowed scope or this card itself marks a branch `Optional / Deferred`.
16b. Do not reinterpret task size, local uncertainty, or partial runtime success as permission to stop after the easiest completed phase, to substitute representative branches for required accounting, or to report the lane as effectively done once the host `.tst` merely executes.
16c. Phase-eligibility rule:
   - downstream required phases remain in scope from intake, but per-slice/per-operation `blocked`/`deferred` accounting becomes final user-visible reporting only after the corresponding happy-path slice or operation is phase-eligible for that downstream phase
   - a slice or operation becomes phase-eligible only when the upstream authored happy-path material for that exact slice/operation is specific enough that the downstream branch could now be authored or accounted for credibly
   - until then, keep that downstream work visible as pending remaining work behind unfinished upstream authoring rather than as if it had already been attempted
16d. Next-safe-action rule:
   - a next safe in-scope action is a deterministic continuation step that fits the approved scope and current posture, can be justified from the allowed evidence ladder, and does not depend on a still-missing user decision or unsafe guess
   - when several such actions exist, prefer the smallest credible continuation that keeps required-phase progress moving
16e. Autonomous continuation rule:
   - once blockers are cleared, keep moving to the next deterministic in-scope cluster, slice, or phase while at least one next safe action remains
   - do not emit the completion artifact merely because exploration, generation, or one cluster's initial structure already proves progress
16f. If some slices are ready for later required phases while others still need upstream happy-path authoring, keep already-ready slices moving when safe and aggregate only true blockers rather than stalling the whole lane because another slice is still pending.
16g. Blocker aggregation rule:
   - treat a local branch as a true blocker only when its next credible step depends on missing auth, missing business inputs, missing prerequisite state, a delegated-branch hard blocker, or another fail-closed dependency gap
   - when batching is possible, return unresolved blockers as one aggregated set while continuing any other slices that still have a next safe action
16h. If no next safe in-scope action remains, or a later required phase still cannot be completed credibly after reasonable continuation, stop as `partial` or `blocked` and report the exact unfinished phase and reason rather than silently narrowing behavior.

### 6.2 Experimental Planning and Cluster Preanalysis
17. Build the selected operation slice from the OpenAPI plus active project context.
18. Build or recompute the transient exploration preanalysis needed for deterministic exploration and cluster formation.
19. Keep that preanalysis transient under `work/`; do not persist exploration-specific structure into `project.yaml`.

### 6.3 Direct Exploration Phase
20. Delegate direct endpoint exploration to Skill 066.
21. Require Skill 066 to produce:
   - one per-operation exploration dossier in a transient ledger
   - one resource/workflow-cluster overlay per approved cluster
   - cluster-local baseline plans, bindings, and ordered `authoringPaths`
   - prerequisite-expansion metadata for any out-of-scope support work
   - preserve/defer recommendations for branches that cannot be promoted into normal executable coverage
   - advisory negative-opportunity and security-targeting hints when evidence supports them
   - normalized mutation-ready request bundles for successful operations
22. Treat the exploration ledger as the evidence contract for later authoring and validation planning.
23. Do not reopen a discovery loop through SOAtest runs once the exploration phase has already produced sufficient evidence.

### 6.4 Experimental Approval Boundary (Interactive Required)
24. In interactive mode, build the pre-write approval artifact using `docs/templates/skill-065-approval-artifact-template.md`.
25. The approval artifact must summarize:
   - selected scope and target asset
   - experimental-lane defaults applied
   - cluster-local happy-path authoring plan
   - any prerequisite-expansion branches that require confirmation before executable authoring
   - `Optional / Deferred`, negative/security accounting plans, and validation intent
   - unresolved request-value/payload approvals that still block writes
   - review items, blockers, or evidence gaps still needing user input
26. If review items remain, require the user to answer them by review id and do not begin writes until those items are resolved and the user explicitly approves the experimental plan.
27. If no review items remain, a simple explicit approval is sufficient.
28. If approval is partial, execute only the confirmed clusters/slices and report the unapproved remainder as deferred rather than silently broadening or shrinking scope.
29. In autonomous mode, skip the approval artifact and proceed once blocker questions are resolved; do not reopen a separate planning/approval interruption later unless a true blocker appears.

### 6.5 Host `.tst` Creation and Template/Source-of-Copy Handling
30. When the branch is broad new-file authoring, create the host `.tst` through `docs/skills/platform/skill-022-tst-create-from-openapi.md`.
31. If subset scope was selected, prune immediately through `docs/skills/structure/skill-047-generated-subset-prune.md`.
32. Treat the REST Clients generated by Skill 022 in this broad new-file lane as the standard unconstrained / None-mode generated branch.
33. Read back the generated root suite, child structure, and generated local environment context.
34. Immediately after create/readback, add or update a `BASEURL` variable in the generated local environment through Skill 064:
   - prefer the confirmed environment-specific base URL already stored in active project context
   - otherwise use the confirmed service-definition base URL for this exact branch
   - do not guess when no single base URL is confirmed
35. Before the generated suite is frozen as source-of-copy, normalize generated template-suite REST Clients through Skill 020 so concrete URLs rooted at that confirmed base become `${BASEURL}` plus the preserved relative path and query.
36. Preserve already-parameterized, relative, or non-matching resource templates as-is, and stop or defer rather than guessing when a generated URL cannot be safely decomposed into confirmed base plus relative remainder.
37. Disable the generated OpenAPI-derived suite through Skill 074 and preserve it as the template/source-of-copy area only after the `BASEURL` normalization step is complete; do not mutate it into executable authored coverage.
38. Create or reuse the sibling top-level suite `AI Generated Tests` through Skill 009.
### 6.5.1 Reference-Only Cross-TST Lookup Boundary
38a. If the agent inspects another `.tst` in the active project or elsewhere in the workspace during this lane, treat that asset as reference evidence only.
38b. Allowed use of such reference lookup is limited to orientation, completeness checks, troubleshooting, or comparison of high-level authored shape; it must not become the mutation source for the new host asset.
38c. Do not copy suites, tests, REST Clients, databanks, assertors, validators, Diff Tools, or other authored nodes from another `.tst` into the new host `.tst`.
38d. In this lane, the approved copy sources are:
   - the disabled generated template suite inside the current host `.tst` for happy-path REST Client source copies
   - already-authored happy-path copies inside the current host `.tst` when deriving negative or security branches
38e. If another `.tst` reveals a useful pattern, recreate that intent through the current host asset's generated template suite plus the owning stable leaves instead of reusing the foreign `.tst` node directly.

### 6.6 Canonical Authored Structure
39. Under `AI Generated Tests`, create one suite per approved resource/workflow cluster through Skill 009.
40. Under each cluster suite, create the canonical child suites in this order:
   - `Happy Path`
   - `Optional / Deferred` when the ledger says such branches exist
   - `Negative Coverage`
   - `Security Coverage`
41. Under `Happy Path`, create `Baseline Snapshot`, `Safe Reference Reads`, and `Lifecycle Flow`.
42. Under `Lifecycle Flow`, create the copyable scenario sub-suites required by the ordered authoring path.
43. Approved prerequisite support steps may live inline inside the ordered happy-path scenario when they are necessary to establish the approved path; do not create a separate support top-level suite unless a later explicit design change authorizes it.
44. Treat those three `Happy Path` children as the initial canonical skeleton, not as placeholders that must survive to the end of the run even when one stayed empty.
45. Treat `Optional / Deferred` as narrow and disabled by default; do not use it as a catch-all merely because a branch is non-restoring, uses different auth, or works against service-provided shared data.

### 6.7 Happy-Path Authoring Mechanics
45. For each happy-path REST step, copy the corresponding normalized unconfigured REST Client from the disabled template suite in the current host `.tst` into the correct target suite through Skill 070; do not substitute a same-looking REST Client copied from another `.tst` asset.
46. Configure each copied REST Client through Skill 020 using the ledger-approved method, endpoint, auth, payload, expected-code, and parameterization decisions, while preserving inherited `${BASEURL}` endpoint parameterization unless the approved branch explicitly needs a different base strategy.
47. If the exploration-backed request requires Basic auth and credentials are available from explicit user input or an approved project secret reference, treat auth wiring as part of Skill 020's native REST Client configuration contract rather than as a separate blocker.
48. After producer reads are configured, attach Data Banks through the payload-appropriate leaf (`Skill 028` for JSON, `Skill 019` for XML) for:
   - whole-payload baseline snapshots when the ledger records whole-payload viability
   - targeted stable extracts
   - runtime-produced ids or selectors needed by later bound steps
49. After consumer or reconciliation reads are configured, attach narrow authoring-integrity Assertors through the payload-appropriate leaf (`Skill 010` for JSON, `Skill 016` for XML) only when they are needed to prove scenario coherence, including:
   - baseline assertions
   - safe-read assertions
   - post-state or restoration assertions
   - non-restoring delta/transition assertions when cleanup is unavailable
49a. Fail closed on weak authoring-integrity assertor targets:
   - do not use generic container-level presence checks merely to prove that a broad envelope node such as `/root`, `/root/data`, or another wrapper has content when stronger stable semantic signals exist
   - prefer targeted stable status fields, identity continuity, echoed business fields, post-state values, or restoration checks that actually prove scenario coherence
   - if the intended check is broad payload-content or schema correctness rather than narrow scenario coherence, leave that work to the Skill 067 validation bundle instead of overloading authoring-integrity assertors
49b. When `hasContentAssertion` is still the correct narrow assertion family, target the smallest semantically meaningful element rather than the broadest container that happens to be non-empty.
50. Treat those databanks and assertors as execution-critical authoring support, not as the full happy-path validation-enrichment phase.
51. Reorder the immediate children of every affected suite through Skill 009 so the physical `.tst` order matches the ledger order exactly.
52. After happy-path authoring is materially complete for a cluster, run a final cleanup pass through Skill 009 delete flow and remove any direct `Happy Path` child suite that remained empty, for example an unpopulated `Lifecycle Flow` in a lookup-only cluster.
52a. Treat a `Happy Path` child suite as empty when it ended with no executable tests, no authored scenario sub-suites, and no other meaningful authored descendants.
52b. Keep only the child suites that actually contain authored coverage or meaningful authored context; do not preserve empty placeholders merely to mirror the initial skeleton.
52c. Do not delete non-empty child suites or intentionally preserved disabled branches that still communicate real authored coverage/state.
52d. Immediately after that cluster-local cleanup pass, re-read the direct `Happy Path` children for that cluster and reconcile the result:
   - if any direct `Happy Path` child suite still satisfies the empty rule from `52a`, delete it before leaving the cluster
   - do not rely on the earlier delete calls alone as proof that cleanup is complete
52e. Do not mark a cluster's happy-path authoring as complete until that post-cleanup readback proves the remaining direct `Happy Path` children are all non-empty or intentionally preserved meaningful branches.
53. Treat the following as normal happy-path variants when assertions are credible:
   - self-restoring owned-fixture lifecycle
   - non-restoring mutable lifecycle where create exists but cleanup does not
   - non-restoring mutable lifecycle that works against service-provided existing data when no create path exists

### 6.8 Preserve / Defer / Omit Rule
52. If an endpoint is in approved scope but exploration still cannot produce a credible invocation shape, do not silently author it as normal executable coverage.
53. Preserve the branch visibly in `Optional / Deferred` when:
   - the endpoint was explicitly user-selected, or
   - the endpoint is cluster-significant enough that omitting it would make the authored result misleading
54. Omit the branch from authored executable coverage when it is peripheral, not explicitly requested, and keeping a disabled placeholder would add clutter without materially improving user understanding.
55. In either preserve or omit cases, report why the branch was not promoted into the normal generated tests.

### 6.9 Negative, Security, and Validation Phases
57. Derive negative and security coverage from the configured happy-path copies in the current host `.tst`, not from the raw template suite or any other `.tst` asset.
57a. When a negative or security branch is derived by copying an already-authored happy-path REST Client or scenario fragment inside the current host `.tst`, inspect the copied producer children immediately and remove inherited happy-path-only enrichment before further branch authoring:
   - remove copied JSON/XML validators, JSON/XML assertors, Diff Tools, and other validation-enrichment tools that are not explicitly part of the derived branch plan
   - remove copied producer-local databanks unless the derived branch intentionally needs them for its own setup, assertions, or reporting
   - reattach only the branch-local tools explicitly justified for that negative or security branch
57b. Before any negative/security accounting or attachment work begins, freeze a coverage-denominator snapshot from authored happy-path readback:
   - enumerate every negative-eligible happy-path slice in stable order
   - enumerate every security-eligible operation in stable order
   - record the exact authored source copy that makes each slice or operation phase-eligible
57c. Treat that snapshot as the accounting denominator for the current pass:
   - use it as the source of truth for per-slice negative accounting and per-operation security accounting
   - if later happy-path authoring materially changes eligibility, refresh the snapshot explicitly before claiming phase completion
58. Before attaching negative branches, perform a required negative-accounting pass across every eligible authored happy-path operation/scenario slice.
59. For each eligible slice, end negative accounting in exactly one state:
   - `delivered`
   - `no-credible-family`
   - `deferred`
   - `blocked`
60. Do not treat the negative phase as satisfied merely because a few strong negatives were created elsewhere; every eligible slice must be explicitly accounted for.
60a. Keep the negative-accounting result visible as a per-slice working summary throughout authoring and completion reporting; do not collapse accounting into an implicit mental tally.
61. Consume any Skill 066 advisory negative-opportunity hints when available, but treat them as advisory input rather than as a rigid contract.
62. Before choosing derivation mode, derive candidate standard-negative families from the exploration ledger, negative-opportunity advisory, contract, request shape, and observed service behavior in this order:
   - explicit non-2xx response families documented in the contract or observed during exploration
   - missing required input
   - null or empty-value violations when materially different from omission
   - enum/type/format/length/range/cardinality violations
   - unsupported or malformed body representation when the request shape makes that mutation meaningful
   - protocol-level negatives such as invalid HTTP method, unsupported media type, or unacceptable `Accept` negotiation when plausible
   - unexpected extra-field/body-shape violations when the contract indicates a closed-world request structure
   - duplicate/conflict state violations when the operation semantics imply uniqueness or conflict handling
   - missing resource when a syntactically valid absent identifier is plausible
63. Require a plausible request mutation or exploration-backed evidence basis for each candidate family; do not create symbolic negative families with no meaningful request-level differentiation.
64. Deduplicate by intended failure mode, keep the strongest justified families, and only then apply a first-pass ceiling of five families per operation/scenario slice; five is a ceiling, not a target.
65. The cap applies per eligible slice, not as a run-wide budget, and it is not permission to stop negative planning once a few negatives exist in other slices.
66. If only weaker evidence exists, keep the smaller justified set rather than manufacturing extra negative families.
67. In autonomous posture, if a slice still has at least one credible negative family, continue with the strongest justified set for that slice rather than surfacing uncertainty about weaker candidate families as a user-facing blocker.
68. If a slice has no credible family after the plausibility filter, record `no-credible-family` or `deferred` and continue; do not escalate that local outcome into a blocker by itself.
69. Reserve `blocked` for true dependency gaps that prevent credible negative authoring for a materially important slice, such as unresolved auth/business inputs or prerequisite state that cannot be synthesized from the allowed evidence ladder.
70. Use derivation modes deterministically:
   - `single-step` when the target case can be expressed against one configured REST Client without prior runtime-created prerequisite state
   - `minimal-prefix` when the target step depends on a short prerequisite setup sequence and the earlier steps are not themselves the subject of the branch
   - `full-sequence` only when the failure or security intent depends on the full scenario context and a smaller derivation would be semantically incorrect or non-credible
71. Prefer the smallest viable derivation: `single-step` over `minimal-prefix`, and `minimal-prefix` over `full-sequence`.
72. Under `Negative Coverage`, use the flattest structure that preserves semantic correctness:
   - `single-step` negatives should place the negative REST Client directly under `Negative Coverage` and encode the operation plus negative case in the test name, for example a deterministic `[neg-<case>]` suffix or equivalent operation-oriented label
   - `minimal-prefix` or `full-sequence` negatives may use one operation/scenario-focused bundle when multiple dependent steps must remain grouped
   - create `Setup`, `Negative Cases`, and `Cleanup` child suites only when that grouped bundle actually needs those branches; do not wrap a lone single-step negative REST Client inside an extra bundle or `Negative Cases` suite
72a. When a negative branch has a credible non-happy-path intent but the exact expected status code is still uncertain, allow one first-pass calibration pattern:
   - author the branch with the narrowest defensible temporary non-happy-path acceptance range justified by the contract or exploration evidence
   - run one focused SOAtest verification of that negative branch and inspect test-execution traffic/results to learn the actual observed status behavior
   - tighten the branch to the exact observed status set and run one verification pass after calibration
   - do not leave the delivered branch at a deliberately broad temporary range once exact behavior is known
73. Before attaching security branches, perform a required security-accounting pass across every eligible authored operation in each approved cluster.
74. Treat an operation as security-eligible when the authored happy-path tree contains at least one configured executable REST Client copy for that exact operation and the copied request/auth/state posture is specific enough to seed a meaningful Penetration Testing Tool branch.
75. For each eligible operation, end security accounting in exactly one state:
   - `delivered`
   - `no-credible-target`
   - `deferred`
   - `blocked`
76. Do not treat the security phase as satisfied merely because one representative branch exists somewhere else in the cluster; every eligible operation must be explicitly accounted for.
77. Consume any Skill 066 advisory security-targeting hints when available, but treat them as advisory input rather than as a rigid contract.
78. For each security-eligible operation, choose one canonical copied happy-path source for the security branch:
   - prefer a `Lifecycle Flow` copy of that exact operation when one exists
   - otherwise use the strongest credible `Safe Reference Reads` copy of that exact operation
   - otherwise use a `Baseline Snapshot` copy of that exact operation only when no better lifecycle/safe-read source exists
   - do not substitute a different operation merely because it lives in the same cluster
79. Prefer the smallest credible copied source that preserves the operation-level request, auth, and state posture relevant to the security branch; do not automatically escalate to a full cluster-sequence copy when a narrower source is sufficient.
80. If the chosen operation needs short prerequisite context to be executable or semantically meaningful, embed only the minimal prerequisite prefix required for that exact security branch.
81. If no clean operation-level seed is justified from the authored happy-path tree or exploration evidence, record `no-credible-target` or `deferred` and continue rather than widening to a representative substitute from elsewhere in the cluster.
82. Reserve security `blocked` for true dependency gaps that prevent credible authoring of a materially important operation-level branch, such as unresolved auth context, unavailable prerequisite state, or an execution path that only exists under an alternate posture the run has not approved.
83. Under `Security Coverage`, organize one security branch per delivered operation, and attach the Penetration Testing Tool under the copied REST Client `Traffic Object` through Skill 062.
84. Treat happy-path validation enrichment as a required phase, not as an optional afterthought.
85. Before delegating to Skill 067, build one advisory handoff packet per authored happy-path slice containing at least:
   - cluster/slice identity and authored scenario path
   - authored producer identity and producer role
   - canonical exploration evidence reference and canonical happy-path request/response pair
   - exploration-backed payload/media-type classification treated as authoritative for validation-family selection
   - candidate expected-content basis and volatility hints
   - schema/service-definition candidates with provenance
   - candidate validator message-mapping basis for validator-capable slices, including the exact authored producer operation identity, the intended OpenAPI path template/method, and the expected response-code basis
   - available databanked bindings from prior steps, including whether an existing producer-local databank already exists
   - candidate cross-step semantic comparisons
   - recommended focused verification scope
   - per-slice readiness/status suggestion (`ready`, `no-tool exception`, `validator-blocked`, or `blocked`)
85a. Enforce validation/source ordering for each delegated slice:
   - classify the semantic response body/payload family first
   - choose the content-validation family (`Assertor` vs `Diff Tool`) second from expected-content basis plus the volatility/business-state heuristic
   - confirm schema/service-definition source for any validator branch third
   - do not let missing schema/service-definition confirmation force whole-response Diff Tool when targeted Assertor remains the semantically correct content-validation family
85b. For validator-capable slices, derive the validator message mapping deterministically before delegation:
   - start from the exact authored producer identity and the operation identity already fixed by happy-path authoring
   - map that identity to the confirmed OpenAPI path template and method for the same operation
   - carry forward the expected response-code basis from the authored slice or the approved calibration context
   - treat observed concrete runtime URLs as confirmation evidence only, not as authority to invent or rename the OpenAPI message path
   - if the authored producer identity, OpenAPI operation template, and expected response-code basis do not converge, return the validator branch as blocked rather than guessing
86. Treat the Skill 066 ledger as advisory input to that packet rather than as a rigid validation contract.
87. Skill 065 keeps ownership of execution-critical databanks and narrow authoring-integrity assertions; Skill 067 owns enrichment-oriented semantic validation, including cross-step comparisons that use earlier databanked values as expected semantic content rather than only request plumbing.
88. Delegate exploration-backed validation planning and attachment to Skill 067 using that handoff packet.
89. For each happy-path slice, validation completion must end in exactly one reported state:
   - `delivered`
   - `delivered-valid-finding`
   - `no-tool exception`
   - `validator-blocked-but-assertor/diff-delivered`
   - `blocked`
90. Keep response-vs-database comparison as an optional later branch rather than part of the mandatory core sequence.
91. When Skill 065 delegates to Skill 058 or Skill 067, preserve the current posture and approval boundaries:
   - autonomous mode must not reopen a separate planning artifact in the delegated branch
   - interactive mode should surface unresolved validation/schema review only within the existing Skill 065 approval/review flow rather than through a separate orphan validation approval artifact
   - unresolved targets or decisions should return as one aggregated blocker set when batching is possible
92. Skill 065 remains the caller-owned user-facing interruption point for aggregated blockers returned from delegated branches and should keep already-ready slices moving when safe instead of stalling the whole lane unnecessarily.

### 6.10 Focused Verification and Bounded Correction
93. Treat Skill 074 disable-run-restore as the default focused-verification mechanism.
94. Capture current disabled states, disable non-target branches, run the intended verification slice, then restore the exact prior disabled states.
95. Use these verification scopes by default:
   - baseline-only verification: enable only the target cluster's `Baseline Snapshot` subtree plus directly chained databanks/assertors required for baseline proof
   - safe-read verification: enable the target `Safe Reference Reads` subtree plus any prerequisite baseline reads whose databanks are required by the safe reads
   - one happy-path scenario verification: enable the one target scenario sub-suite plus the minimum prerequisite baseline/safe-read producers required by its databanks/assertors
   - one negative target verification: enable only the selected negative REST Client or grouped negative bundle; when the branch uses grouped structure, include its local `Setup`, `Negative Cases`, and `Cleanup` children when present
   - one security branch verification: enable only the selected security branch and the local copied prerequisite steps embedded in that branch
   - cluster-level happy-path verification: enable the target cluster's `Happy Path` and keep `Optional / Deferred`, `Negative Coverage`, and `Security Coverage` disabled unless the user explicitly requested a broader run
96. Use `soatestOptions.testNames` only as a fallback when names are already sufficiently isolated and the disable-run-restore sandbox adds no value.
97. Treat the first SOAtest run in this lane as verification, not baseline discovery.
98. If verification fails even though direct exploration succeeded:
   - collect the normal results/readback/traffic triad
   - compare persisted state against the canonical exploration bundle and approved validation plan
   - allow at most one automatic correction attempt only when the evidence shows a dominant SOAtest-side persistence/configuration defect
   - otherwise stop as blocked or partial
99. Do not execute copied security suites by default during interim verification unless the user explicitly asked for security execution.

### 6.11 Completion Reporting
100. Report the final result using `docs/templates/skill-065-completion-artifact-template.md`.
101. Completion reporting must state:
   - that the lane was experimental and explicit opt-in
   - which clusters/slices were delivered, partial, blocked, or deferred
   - that the design baseline was exploration-backed and the SOAtest run was verification-focused
   - the negative-accounting summary for every eligible slice, including where negative coverage was delivered versus `no-credible-family`, deferred, or blocked
   - the security-accounting summary, including where security coverage was delivered versus `no-credible-target`, deferred, or blocked
   - any validation slices that ended as `delivered-valid-finding`, including whether the finding came from schema/content validation rather than authoring drift
   - any notable negative-behavior findings that materially affected negative accounting or expected-code calibration, such as invalid or absent-resource probes that still returned `2xx` or empty-success-like bodies
101a. Distinguish phase-eligible outcomes from pending downstream required work:
   - do not label downstream negative/security/validation/verification slices `blocked` or `deferred` merely because their happy-path prerequisites were still in progress when the run stopped
   - when the stop happened earlier, keep those later required phases visible in required-phase status and remaining/follow-up items as pending work rather than as if they were already attempted
101b. When a run stops with a mix of delivered work and unresolved blockers, report both:
   - the phase-eligible slices or operations that were actually delivered or truly blocked/deferred
   - the later required work that remained phase-pending because the run stopped earlier
101c. Before reporting the lane as `complete`, run one final authored-structure reconciliation across all delivered clusters:
   - re-read each cluster's direct `Happy Path` children
   - verify that no direct `Happy Path` child suite still satisfies the empty rule from `52a`
   - if any empty direct `Happy Path` child suite remains, treat cleanup as unfinished required work, delete/reconcile it when safe, and otherwise report the lane as `partial` rather than `complete`
102. Do not report the lane as complete or emit a terminal completion artifact while a next deterministic in-scope action still remains after blockers are cleared, even if exploration, host `.tst` generation, or template-suite restructuring already succeeded.
103. If later phases cannot proceed after reasonable continuation and no next safe action remains, report `partial` or `blocked` and name the unresolved clusters, review items, or delegated-branch blockers rather than silently ending the run early.
104. After completion, optional follow-up offers may include exploration-backed response-vs-database comparison, a broader cluster-level verification pass, or a full `.tst` execution that includes security coverage when the user explicitly wants that broader run.

## 7) Validation
- The experimental branch never activates without explicit opt-in.
- The lane remains REST/OpenAPI-only in v1.
- Skill 065 owns its own posture-first intake, naming gate, and exploration-backed request-value/payload gate rather than relying on implicit Skill 033 behavior.
- Interactive mode presents the operation catalog before all-vs-subset decisions and does not write before explicit approval of the exploration-backed plan.
- Autonomous mode remains blocker-only once minimally viable inputs are resolved and does not reopen generic planning interruptions after that point.
- Direct exploration evidence is gathered before SOAtest writes begin.
- The exploration ledger remains transient under `work/`.
- Existing stable leaves remain authoritative for actual SOAtest mutation and execution mechanics.
- The generated OpenAPI suite is normalized around a local `BASEURL` variable before it is preserved as the disabled template/source-of-copy area rather than converted directly into final executable coverage.
- Lookup of other `.tst` assets, when used at all, remains reference-only and never becomes the copy or mutation source for the new host asset.
- Happy-path REST Client source copies come only from the disabled generated template suite in the current host `.tst`.
- `AI Generated Tests` is the canonical authored sibling root.
- Cluster suites follow the canonical child structure `Happy Path`, optional narrow `Optional / Deferred`, `Negative Coverage`, and `Security Coverage`.
- Empty direct `Happy Path` child suites that stayed unpopulated after authoring are deleted in the final cleanup pass rather than preserved as empty placeholders.
- Cluster-local cleanup is readback-verified before the agent leaves the cluster, and final completion is gated on a run-wide reconciliation that proves no empty direct `Happy Path` child suites remain.
- Credentialed Basic-auth slices are not blocked solely because generated clients initially lack auth; the lane delegates native auth wiring for the generated broad-authoring branch to Skill 020 when supported.
- Approved secret references may be consumed for supported exploration/auth-write branches without forcing an environment/reference concealment detour.
- Standard-negative/security/validation phases are derived from exploration-backed happy-path evidence rather than from a separate pre-attachment SOAtest discovery run.
- Negative/security derivation removes inherited happy-path validation/enrichment children from copied producers unless those children are explicitly needed by the derived branch.
- Before negative/security accounting is considered complete, the lane freezes and uses an explicit happy-path-derived coverage denominator snapshot rather than relying on an implicit mental inventory.
- Security planning uses required per-operation security-accounting outcomes and exact-operation target selection rather than representative per-cluster substitution.
- Happy-path validation enrichment is required per slice and uses a caller-owned Skill 65 -> Skill 67 handoff packet rather than a loose delegated suggestion.
- Validator-capable slices carry an explicit message-mapping basis from Skill 065 into Skill 067 so OpenAPI validator mapping is derived from authored producer identity plus the confirmed operation template rather than semantic guesswork.
- Narrow authoring-integrity assertors prefer semantically meaningful fields or relationships over broad wrapper-level `hasContent` checks.
- Required Skill 065 phases follow the global mandatory-phase-integrity rule; task size, uncertainty, or partial phase success are not permission to skip later required phases, and unfinished required phases must be reported as `partial` or `blocked` explicitly once no next safe in-scope action remains.
- Downstream required phases that are not yet phase-eligible stay visible as pending remaining work rather than being reported as local `blocked`/`deferred` outcomes merely because upstream authoring is unfinished.
- The lane keeps already-actionable slices moving and aggregates only true blockers; one blocked slice is not by itself permission to stop the whole run while another next safe in-scope action still exists.
- Validation completion may end as `delivered-valid-finding` when a correctly configured validator/assertor/diff branch surfaces a genuine service defect; such findings are not permission to delete the tool or reinterpret the failure as misconfiguration by default.
- Standard-negative planning uses an explicit family-selection heuristic, required per-slice negative-accounting outcomes, plausibility filter, deduplication rule, and cap/ceiling behavior inside this card.
- Single-step negatives default to directly named operation-oriented REST Clients under `Negative Coverage`; extra bundle and `Negative Cases` hierarchy is reserved for grouped `minimal-prefix` or `full-sequence` negatives that truly need it.
- The five-family ceiling is applied late, per eligible slice, and is not a run-wide stopping condition.
- Autonomous negative uncertainty defaults to local `no-credible-family` or `deferred` accounting rather than user-facing blocker interruption unless a true dependency gap exists.
- Security planning uses lifecycle-preferred source selection for each exact operation and falls back to safe-read or baseline copies of that same operation only when no stronger lifecycle candidate exists.
- Autonomous security uncertainty defaults to local `no-credible-target` or `deferred` accounting rather than broadening to a different representative operation or surfacing a blocker unless a true dependency gap exists.
- Delegated branches preserve posture and return aggregated blockers when batching is possible.
- Focused verification defaults to Skill 074 sandboxing rather than name-filtered execution.
- Post-authoring correction is bounded to one evidence-backed persistence/configuration repair attempt.
- Completion is not reported early while approved downstream experimental phases still remain.
- Final reporting includes negative-accounting outcomes so undercoverage is explicit rather than implied.

## 8) Failure Modes
- Experimental opt-in is assumed from vague wording and the branch is activated incorrectly.
- SOAP or non-OpenAPI authoring is forced into the experimental lane.
- The lane still behaves as if Skill 033 owns posture resolution, naming, or request-value/payload approval.
- Direct endpoint exploration is treated as interchangeable with stable execution-backed evidence outside this explicit lane.
- Exploration evidence is not captured clearly enough to drive later authoring and validation.
- The lane drifts back into a flat per-operation CRUD structure instead of the canonical cluster-local authored shape.
- The generated template suite is frozen before `BASEURL` normalization, causing later copies to keep concrete host URLs.
- The generated template suite is mutated directly instead of being preserved as disabled source-of-copy.
- The lane guesses a `BASEURL` mapping or rewrites non-matching generated URLs into `${BASEURL}` even though safe decomposition was not proven.
- `Optional / Deferred` becomes a catch-all bucket for ordinary non-restoring or service-provided-data branches.
- Empty `Happy Path` child suites are left behind even though they never received viable authored content, making the final cluster structure look broader than it is.
- The lane performs cleanup deletes but fails to re-read the cluster or run-wide authored structure, so empty direct `Happy Path` child suites survive into reported completion.
- Another `.tst` is treated as a source-of-copy for authored suites, tests, or tools instead of remaining reference-only context.
- The lane invents a parallel mutation stack instead of delegating to existing stable leaves.
- Copied negative or security branches retain inherited happy-path validators, assertors, diff tools, or databanks that were not intentionally part of the derived branch.
- A few negatives in one cluster are treated as sufficient without accounting for the remaining eligible slices.
- The negative/security denominator is never frozen from happy-path readback, leaving coverage breadth to an implicit mental tally.
- The five-family ceiling is misread as a run-wide budget or as permission to stop early once some negatives exist elsewhere.
- Ordinary uncertainty about a weak negative family is surfaced as a user-facing blocker in autonomous mode instead of being recorded as `no-credible-family` or `deferred`.
- Single-step negatives are buried under unnecessary extra suite hierarchy even though no grouped prerequisite flow was required.
- A negative branch keeps a deliberately broad temporary status-code range even after focused calibration established the exact expected behavior.
- One representative security branch per cluster is treated as sufficient without accounting for the remaining eligible operations.
- Security authoring widens to a different operation in the same cluster instead of keeping exact-operation security targeting or recording `no-credible-target`/`deferred`.
- Lifecycle-flow security seeds are skipped even when they exist, producing weaker operation-level security branches than the lane intended.
- Interactive request/value approvals are left implicit, causing exploration-backed write drift.
- Delegated branches reopen separate planning/approval loops and break the agreed autonomous posture.
- A correctly configured validator that reports a genuine schema/content defect is deleted, disabled, or downgraded merely to make verification pass.
- Task size or local uncertainty causes the lane to skip required downstream phases or reinterpret partial progress as effective completion.
- Skill 065 omits the validation handoff packet, leaving Skill 067 to rediscover databank context, expected-content candidates, or verification scope from scratch.
- Happy-path validation is reported complete even though a slice never reached `delivered`, `no-tool exception`, `validator-blocked-but-assertor/diff-delivered`, or `blocked`.
- The lane reports `blocked-auth` for generated REST Clients even though Skill 020 supports the validated Basic-auth update path for the generated broad-authoring branch.
- The lane detours into referenced-environment or other concealment workarounds solely to avoid writing supported auth values into the generated `.tst`.
- Focused verification falls back to `testNames` even though Skill 074 sandboxing was practical.
- A failed focused verification run reopens an unbounded rediscovery loop.
- Completion is reported immediately after exploration or generation even though approved negative/security/validation/verification work still remains.
- Security branches are pulled into interim verification runs even though no explicit security execution was requested.

## 9) Safety / Rollback
- Treat the experimental lane as write-capable and side-effect-capable because direct exploration may include bounded state-changing CRUD probes.
- Keep all exploration artifacts and ledgers transient under `work/`.
- Delegate object-level rollback to the downstream stable leaves used for each mutation surface.
- Stop rather than guessing when auth, business data, schema-source confirmation, or runtime evidence remain insufficient.

## 10) Reuse Notes
- This is the top-level explicit opt-in owner for the experimental lane.
- Use stable Skill 033 when experimental opt-in is absent.
- Use Skill 066 for the stateful direct-exploration rules, evidence contract, and advisory negative/security targeting surfaces.
- Use Skill 067 for exploration-backed validation planning and attachment.
- In this lane, immediate post-generation template normalization is caller-owned: Skill 064 supplies the generated local `BASEURL` environment variable, Skill 020 rewrites matching generated REST Client URLs to `${BASEURL}` plus relative path/query, and Skill 022 remains generation-only.
- Other `.tst` assets may be inspected as reference-only evidence, but they are not approved source-of-copy inputs for this lane.
- Keep stable Skills 033, 057, and 058 unchanged as the default operator-facing branch outside this lane.

## 11) Prompt Sequence Templates
Use these condensed sequences to keep the experimental lane self-contained and posture-aware.
### 11.1) Interactive Path
1. If posture is unresolved: `Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the experimental plan together? Just say 'autonomous' or 'auto', or 'interactive'.`
2. After explicit experimental opt-in and source confirmation, present the operation catalog first, then ask: `Here are the operations I found. Do you want all of them or a subset?`
3. `Do you want me to also include standard negative tests?`
4. `Do you want me to add security tests?`
5. `What should I name the `.tst` file?` (omit when target host/name is already resolved)
6. For body-bearing operations: `For requests that include a body or concrete values, do you want me to show the exploration-backed request bundle for approval, proceed without that approval, or would you like to provide values yourself? You can say: approval, no approval, or manual.`
7. `Reply to any review items by id if they remain, then approve the artifact if you want me to proceed.`
### 11.2) Autonomous Path
1. If posture is unresolved: `Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the experimental plan together? Just say 'autonomous' or 'auto', or 'interactive'.`
2. Once the experimental lane is confirmed and the OpenAPI source is usable, default to full scope unless the user already constrained the operation slice.
3. If a host asset is still unresolved, default to deterministic `.tst` naming rather than interrupting solely for name approval.
4. Keep standard negatives, security, and exploration-backed validation on by default unless the user constrained the run.
5. Accept exploration-backed request bundles for body-bearing happy-path operations once auth/business blockers are resolved.
6. Proceed without a pre-write approval artifact once blocker questions are cleared.
7. Record local negative-accounting outcomes that do not reflect true dependency gaps as `no-credible-family` or `deferred` rather than surfacing them as blockers.
8. If delegated branches later return unresolved blockers, interrupt once with the aggregated blocker set rather than surfacing one blocker at a time.
### 11.3) Agentic Context Adaptation
In non-conversational or agentic execution contexts:
- skip questions the user has already resolved
- preserve the same posture rules:
  - interactive mode keeps explicit approval of the exploration-backed approval artifact before writes
  - autonomous mode asks only blocker questions and otherwise continues execution
- never bypass mandatory gates:
  - explicit experimental opt-in
  - usable REST/OpenAPI source
  - `.tst` naming resolution through either explicit choice or authorized defaulting
  - exploration-backed request-value handling through either interactive approval/manual input or autonomous acceptance
