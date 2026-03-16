# Skill 066: Experimental Direct API Exploration Evidence Policy

## 1) Skill Name
Bounded direct API exploration and transient evidence policy for the explicit opt-in experimental broad-authoring lane.

## 2) Objective
Define how the experimental lane performs direct endpoint exploration before SOAtest authoring: how request hypotheses are built, how resource/workflow clusters are identified, how state-changing probes and reconciliation are bounded, what evidence is considered legitimate, how transient exploration artifacts are stored, how bounded additional exploration may be used to improve negative-opportunity or security-targeting advice, and how successful findings are normalized into a stateful cluster ledger plus mutation-ready request bundles for downstream SOAtest authoring.

## 3) Scope
- In scope:
  - direct HTTP exploration for REST/OpenAPI operations in the explicit opt-in experimental lane
  - request-hypothesis generation from contract, project context, user values, and same-run evidence
  - resource/workflow clustering and cluster-boundary heuristics
  - method safety classes for observational, state-advancing, and destructive probes
  - bounded retry/attempt rules
  - CRUD-oriented reconciliation after state-changing probes
  - transient exploration-ledger shape and evidence-legitimacy rules
  - stateful cluster overlay with baseline plans, bindings, role/semantic tags, and ordered `authoringPaths`
  - subset-scope prerequisite expansion metadata
  - normalized request-bundle production for downstream SOAtest mutation owners
  - advisory negative-opportunity hints for downstream Skill 065 negative accounting
  - advisory security-targeting hints for downstream Skill 065 security accounting
  - bounded additional exploration for negative-opportunity or security-targeting discovery when existing evidence is insufficient
  - outcome classification for explored operations and branches
- Out of scope:
  - activating direct exploration as a default baseline outside the experimental lane
  - guessing credentials or inventing secrets
  - unbounded trial-and-error probing
  - replacing the lower-layer SOAtest mutation owners
  - durable storage of exploration-derived OpenAPI structure in project records

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`

## 4) Inputs
- Required:
  - selected operation slice
  - OpenAPI contract details for the selected operations
  - active project context relevant to the branch
  - explicit user-provided request values or constraints when available
- Optional:
  - transient exploration preanalysis artifact for reproducibility
  - already-learned same-run identifiers or reusable successful values

## 5) Preconditions
- The branch is inside the explicit opt-in experimental lane.
- The target source is REST/OpenAPI.
- The API branch has passed capability preflight for the needed read/write calls.
- Credentials are either explicitly available, resolvable from an approved project secret reference, or the operation can be classified `blocked-auth`; do not guess credentials.

## 6) Procedure
### 6.1 Inputs and Outputs
1. Treat the exploration phase outputs as:
   - one per-operation exploration dossier in a transient ledger under `work/`
   - one resource/workflow-cluster overlay per approved cluster
   - a selected canonical happy-path request when one reaches `usable happy path`
   - response-shape, volatility, assertion-design hints, and negative-opportunity/security-targeting hints derived from the successful response when justified
   - final blocked/exhausted/mismatch classifications when no usable happy path is found
   - one normalized mutation-ready request bundle per successful operation
2. Treat the exploration ledger as the evidence contract for later authoring and validation; downstream phases should consume it rather than rediscovering the same facts through unfinished SOAtest tests.

### 6.2 Deterministic Cluster Formation
3. Do not equate OpenAPI tags with clusters mechanically.
4. Apply these cluster-boundary heuristics:
   - prefer one cluster when operations share a stable resource identity and clearly participate in one state story, even if the OpenAPI uses multiple tags
   - split clusters when operations depend on different primary identities or represent different lifecycle owners, even if they share a tag
   - treat lookup/supporting endpoints as their own read-only cluster when they primarily provide selector values or context for other clusters rather than owning the main mutable state
   - record cross-cluster dependencies explicitly rather than forcing dependent operations into the same cluster just because they exchange ids or selector values
   - use hybrid-workflow clustering when a group of operations forms one user-visible workflow but contains multiple authoring paths with different safety or privilege postures
   - reserve operational/admin endpoints that mutate environment-wide state for separate non-default clusters rather than merging them into business-resource clusters
   - when membership remains ambiguous after identity and dependency analysis, prefer the narrower cluster boundary and express the relationship through explicit external bindings
5. Within each cluster, prefer this exploration posture order unless stronger same-run evidence justifies a different sequence:
   - safe baseline/lookup reads first
   - safe reference reads next
   - fixture create and fixture-bound mutation/read steps after the needed safe baseline evidence exists
   - cleanup/delete/terminal destructive steps last
6. Preserve stable OpenAPI order among operations that share the same posture role.

### 6.3 Request-Hypothesis Generation
7. Before the first live probe for an operation, build one request hypothesis containing:
   - method and path
   - resolved base URL candidate
   - auth/header hypotheses
   - path/query inventory with candidate values and provenance
   - request-body schema summary when applicable
   - candidate payload object when applicable
   - expected response families from the contract
8. Candidate-value precedence is:
   1. explicit user-provided value
   2. approved project-context value or durable project data/reference
   3. operation-local OpenAPI examples/defaults/enums
   4. reusable values learned from earlier successful exploration in the same run when meaning is clearly compatible
   5. synthesized schema-conformant placeholder values
Auth note:
- for credential-bearing auth hypotheses, approved project secret references are valid high-precedence sources; resolve plaintext values from the referenced gitignored location when needed
9. Never synthesize credentials, secrets, or auth tokens.
10. If auth remains unresolved after checking explicit input and approved secret references, classify the operation `blocked-auth` rather than probing blindly.

### 6.4 Method Safety Classes and Bounded Attempt Loop
11. Treat `GET`, `HEAD`, and `OPTIONS` as observational probes once base URL and auth are sufficiently resolved.
12. Treat `POST`, `PUT`, and `PATCH` as state-advancing probes:
   - probing is allowed when the selected operation is in scope and the request structure is meaningful enough to attempt
   - incidental state change is accepted as normal exploration evidence, not a default failure condition
   - every successful or partially successful state-changing probe must be followed by reconciliation
13. Treat `DELETE` as destructive probing with stricter target-resolution requirements:
   - probing is allowed only when target identity is concrete enough to avoid blind destructive guessing
   - post-delete reconciliation is mandatory whenever the contract/runtime makes it possible
14. For each operation, use a bounded attempt loop:
   - Attempt 0: hypothesis assembly only
   - Attempt 1: best initial request from the current evidence ladder
   - Attempt 2: allowed only when Attempt 1 produced narrowing evidence
   - Attempt 3: allowed only when Attempt 2 produced fresh narrowing evidence
   - no fourth attempt
15. Blind retries are forbidden.
16. Each retry must cite the new evidence that changed the request.

### 6.5 Reconciliation After State Change
17. After any successful or state-changing write probe, run reconciliation whenever the contract/runtime makes it possible.
18. Preferred reconciliation order:
   1. direct `GET` of the returned or inferred resource id/path
   2. collection/list read that can confirm the new current state
   3. another contract-compatible existence/update/delete confirmation call
19. Record both the immediate mutation response and the reconciled post-state in the same attempt record.
20. For `POST`, treat the immediate create response as the primary authored `POST` baseline unless it is too sparse to support meaningful downstream design.
21. For self-restoring scenarios, record whether exact whole-payload return-to-baseline appears viable; if viability is uncertain, fail closed to targeted reconciliation rather than assuming exact whole-payload comparison will be reliable.

### 6.6 Outcome Classification
22. Each explored operation ends in exactly one primary state:
   - `usable happy path`
   - `blocked-auth`
   - `blocked-input`
   - `probe-exhausted`
   - `negative-only-evidence`
   - `spec-runtime-mismatch`
23. State change is not a primary state by itself; it is an attempt annotation paired with reconciled post-state.
24. When several attempts succeed, choose the canonical happy-path request by:
   1. highest-precedence input provenance
   2. simplest successful request shape
   3. response and reconciled post-state that are semantically complete enough for validation design
24a. Reuse existing exploration evidence first when forming negative-opportunity advice for downstream Skill 065 negative accounting.
24b. If the existing exploration pass does not provide enough evidence to support a clean negative-opportunity advisory for a materially relevant slice, allow bounded additional exploration targeted specifically at clarifying likely negative families, prerequisite needs, or mutation-safety notes.
24c. Keep that additional exploration narrow:
   - do not widen into an exhaustive negative matrix
   - do not turn the lane into open-ended fuzzing or repeated trial-and-error probing
   - do not expand into unrelated operations merely to hunt for more negatives
   - continue to respect the same method-safety classes, reconciliation rules, and bounded-attempt discipline defined in this card
24d. Record only evidence-backed negative-opportunity hints, such as likely candidate families, prerequisite needs, autonomous-safety notes, and omit/defer indicators.
24e. The negative-opportunity overlay remains advisory; Skill 065 still owns final negative-accounting outcomes and breadth decisions.
24f. Reuse existing exploration evidence first when forming security-targeting advice for downstream Skill 065 security accounting.
24g. If the existing exploration pass does not provide enough evidence to support clean operation-level security-targeting advice for a materially relevant operation, allow bounded additional exploration targeted specifically at identifying a credible source copy, prerequisite/setup sensitivity, auth-context constraints, or omit/defer indicators.
24h. Keep that additional exploration narrow:
   - do not widen into broad pen testing, open-ended fuzzing, or repeated trial-and-error probing
   - do not expand into unrelated operations merely to find a representative security branch for the cluster
   - continue to respect the same method-safety classes, reconciliation rules, and bounded-attempt discipline defined in this card
24i. Record only evidence-backed security-targeting hints, such as the exact operation, lifecycle-preferred source suggestions, prerequisite/setup notes, alternate-auth concerns, and `no-credible-target`/defer indicators.
24j. The security-targeting overlay remains advisory; Skill 065 still owns final security-accounting outcomes and exact-operation source selection.

### 6.7 Cluster Overlay Contract
25. Keep the per-operation dossier as the source of truth for request/response evidence, and add a compact cluster overlay with this deterministic shape:
   - `clusterId`, `label`, `pathScope[]`, `summaryMode`
   - `identityModel`
   - `baselinePlan`
   - `operationRoles{}`
   - `operationSemantics{}`
   - `bindings`
   - `authoringPaths`
   - `negativeOpportunityAdvisory[]` when evidence supports it
   - `securityTargetAdvisory[]` when evidence supports it
26. `identityModel` should capture at least:
   - `responseIdentityFields[]` with `kind` (`primary`, `alternate`, `foreign`)
   - `routeBindingFields[]`
   - `alternateLocators[]`
   - `foreignKeyInputs[]`
27. `baselinePlan` should capture at least:
   - `snapshotCandidates[]`
   - `targetedExtractions[]`
   - whole-payload viability only as a lightweight record, not as duplicated response bodies
28. Record whole-payload viability as `true` only when all of these hold:
   - the baseline read is a collection or aggregate response whose ordering already appears stable without extra normalization work
   - volatile fields do not dominate the payload
   - payload size remains practical for databanking and later assertion authoring
   - the expected happy-path posture is self-restoring, so exact return-to-baseline is meaningful
29. If any viability condition fails or remains uncertain, fail closed to targeted reconciliation rather than exact whole-payload comparison.
30. Use these controlled operation role tags:
   - `baselineCollectionRead`
   - `safeReferenceRead`
   - `lookupSeedRead`
   - `aggregateStateRead`
   - `fixtureCreate`
   - `fixtureMutation`
   - `fixtureDelete`
   - `bulkCleanup`
   - `reconciliationRead`
   - `sharedResourceMutation`
   - `privilegedMutation`
   - `negativeSeed`
   - `securitySeed`
31. Use these controlled semantics tags:
   - `safeAgainstSharedBaseline`
   - `parameterizedFromBaseline`
   - `requiresFreshFixture`
   - `safeWhenBoundToOwnedFixture`
   - `selfRestoringWhenPairedWithCleanup`
   - `leavesResidualStateWithoutCleanup`
   - `sharedResourceOnly`
   - `borrowAndRestoreCandidate`
   - `terminalDestructiveAgainstSharedBaseline`
   - `deferredFromV1DefaultAuthoring`
   - `requiresAlternateAuthContext`
32. `bindings` should capture at least:
   - `internalValueBindings[]`
   - `externalValueBindings[]`
   - `prerequisiteEdges[]`
   - `clusterDependencies[]`
   - `preferredReconciliationOrder[]`
33. Use these controlled reconciliation-order tokens:
   - `directOwnedResourceRead`
   - `collectionPresenceCheck`
   - `aggregateStateCheck`
   - `bulkCleanupConfirmation`
   - `finalCollectionReturnToBaselineCheck`
   - `sharedResourceReadback`
   - `restoreOriginalValuesIfSupported`
33a. When `negativeOpportunityAdvisory[]` is present, each advisory entry should capture only evidence-backed hints such as:
   - the authored slice or operation it applies to
   - likely candidate negative families
   - prerequisite needs or setup sensitivity
   - mutation-safety notes for autonomous authoring
   - omit/defer indicators when the strongest candidates remain weak or non-credible
   - references back to the exploration evidence that justified the hint
33b. When `securityTargetAdvisory[]` is present, each advisory entry should capture only evidence-backed hints such as:
   - the exact authored operation it applies to
   - preferred security-seed candidates, with lifecycle-flow preference when justified
   - prerequisite or setup sensitivity needed to make that exact operation a credible security branch
   - auth-context or alternate-posture concerns
   - `no-credible-target` or defer indicators when the strongest candidates remain weak or misleading
   - references back to the exploration evidence that justified the hint

### 6.8 Authoring Paths and Prerequisite Expansion
34. `authoringPaths` are the actionable sequencing contract for downstream authoring.
35. Use these controlled path names in the first experimental version:
   - `read-safe-only`
   - `owned-fixture-lifecycle`
   - `partial-crud-mutation`
   - `shared-resource-borrow-restore-later`
   - `hybrid-safe-core`
36. For each cluster, emit:
   - one ordered `defaultPath`
   - zero or more `optionalPaths[]` with `entryConditions[]`
   - zero or more `deferredPaths[]` with `deferredReasons[]`
37. `authoringPaths.defaultPath.operationIds[]` and any optional/deferred path operation lists should be explicitly ordered by the exploration ledger itself; dependency edges remain supporting rationale and validation for that order, not the only sequencing contract.
38. When the user selects only a subset of operations, exploration may widen beyond that subset only for bounded prerequisite discovery and evidence gathering.
39. For any discovered out-of-scope support branch, record at least:
   - `scopeOrigin`: `userSelected`, `discoveredPrerequisite`, or `discoveredOptional`
   - `supportType`: `readOnlyPrerequisite` or `stateChangingPrerequisite`
   - `supportsOperationIds[]`
   - `authoringApprovalRequired`
40. Read-only prerequisite discovery is lower-friction than state-changing prerequisite discovery, but both must still be surfaced when they would expand authored executable scope beyond the user's original scope.
41. If exploration cannot determine a credible invocation shape for an in-scope endpoint, record the deferred signal and enough significance metadata for Skill 065 to apply the preserve/defer/omit rule deterministically.

### 6.9 Mapping into SOAtest Authoring
42. Produce one normalized request bundle per successful operation containing:
   - endpoint/base URL target
   - method
   - headers except tool-owned `Content-Type`
   - path/query values
   - body payload and content type when applicable
   - expected happy-path status-code set
43. Downstream mutation should route through the existing stable owners:
   - Skill 020 for unconstrained/None-mode REST Clients
   - Skill 059 for constrained same-operation clients
44. The cluster overlay plus normalized request bundles are the canonical handoff contract for Skill 065.

## 7) Validation
- Request hypotheses are built before live probes begin.
- Candidate-value precedence remains deterministic.
- No credential guessing occurs.
- Approved secret references may be consumed for auth hypotheses without being mistaken for missing-auth blockers.
- Blind retries are forbidden.
- Write probes are reconciled whenever possible.
- Ledger artifacts stay transient under `work/`.
- Exploration emits both per-operation dossiers and a stateful cluster overlay.
- Exploration may emit advisory negative-opportunity hints for downstream Skill 065 negative accounting when evidence justifies them.
- Exploration may emit advisory security-targeting hints for downstream Skill 065 security accounting when evidence justifies them.
- Bounded additional exploration for negative-opportunity or security-targeting discovery is allowed only when it materially improves that advisory and remains narrow.
- `authoringPaths` remain the authoritative sequencing contract for downstream authoring.
- Subset-scope prerequisite expansion is explicit rather than silently widening executable coverage.
- Only legitimate exploration evidence is allowed to influence downstream authoring and validation.

## 8) Failure Modes
- OpenAPI tags are treated as cluster boundaries without identity/dependency analysis.
- Same-run exploration values are reused across incompatible fields.
- Direct probes proceed even though auth is unresolved.
- A branch is marked `blocked-auth` even though an approved secret reference exists and could have been resolved.
- Exploration opens an environment/reference concealment workaround solely to avoid later supported plaintext auth materialization.
- State-changing methods are treated as fail-open with no reconciliation.
- `DELETE` probes proceed without strong target identity.
- The retry budget is exceeded or used for blind guesses.
- The ledger records status codes but not enough request/provenance context to explain what succeeded.
- The ledger stops at per-operation notes and fails to emit ordered cluster-local `authoringPaths`.
- Negative-opportunity advice is omitted for materially relevant slices even though bounded additional exploration could have clarified it.
- Negative-opportunity discovery expands into open-ended negative probing or broad fuzzing instead of staying targeted and bounded.
- Weak or unsupported negative hints are emitted without evidence, safety notes, or omit/defer guidance.
- Security-targeting advice is omitted for materially relevant operations even though existing or bounded additional exploration could have identified a clean exact-operation seed.
- Security-targeting advice widens to a representative operation elsewhere in the cluster instead of identifying the exact operation seed or emitting `no-credible-target`/defer guidance.
- Additional security-targeting exploration drifts into broad pen testing, open-ended fuzzing, or unrelated-operation probing.
- Exploration-derived OpenAPI structure is persisted durably into project context.
- Downstream phases author from a non-canonical exploration attempt.

## 9) Safety / Rollback
- This policy authorizes bounded direct exploration, including some state-changing requests, only inside the explicit experimental lane.
- The main safety controls are:
  - strict opt-in lane boundary
  - no credential guessing
  - bounded attempts
  - reconciliation after write probes
  - transient evidence storage under `work/`
- Rollback of authored SOAtest assets remains the responsibility of the downstream mutation owners.

## 10) Reuse Notes
- This card is a cross-cutting policy/evidence owner, not a first-choice operator-facing route.
- Skill 065 is the expected caller.
- Skill 067 consumes the legitimacy and evidence rules defined here.
- Skill 065 may consume the advisory negative-opportunity and security-targeting overlays when performing required negative and security accounting.
- Keep stable execution-backed evidence rules unchanged outside this explicit lane.
