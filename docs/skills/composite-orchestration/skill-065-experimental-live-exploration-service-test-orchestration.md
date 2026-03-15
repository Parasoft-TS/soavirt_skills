# Skill 065: Experimental Live-Exploration Service-Test Orchestration (Composite)

## 1) Skill Name
Explicit opt-in experimental broad service-test authoring through live API exploration before SOAtest authoring.

## 2) Objective
Provide an additive experimental broad-authoring lane for REST/OpenAPI service testing that replaces the stable generation-first discovery loop with an exploration-first sequence: analyze the selected service definition, perform bounded direct endpoint exploration, capture transient exploration evidence, author SOAtest assets from that evidence, and run only the minimum SOAtest verification needed to confirm persisted behavior and approved chained tooling.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting scope, opt-in, and execution posture.

## 3) Scope
- In scope:
  - explicit opt-in experimental broad authoring for REST/OpenAPI service definitions
  - reuse of Skill 033-style intake for posture, scope, subset selection, negative/security intent, and optional DB follow-on intent
  - exploration-first orchestration before SOAtest writes
  - delegation to Skill 066 for direct API exploration rules and evidence legitimacy
  - delegation to existing generation, pruning, client-mutation, validation, and security leaves after exploration evidence is ready
  - evidence-backed standard-negative derivation and security-branch sequencing
  - explicit interactive approval artifact before writes
  - autonomous/no-interrupt execution posture after explicit user opt-in
  - bounded post-authoring correction when focused SOAtest verification reveals persisted authoring/configuration mismatch
  - concise completion reporting for the experimental lane
- Out of scope:
  - stable default routing for broad authoring when experimental opt-in is absent
  - SOAP or non-OpenAPI broad authoring
  - replacing atomic mutation/execution owners
  - weakening stable Skills 033, 057, or 058 in place
  - turning direct endpoint exploration into a general-purpose baseline outside this explicit lane

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`
- Additive:
  - `docs/skills/platform/skill-022-tst-create-from-openapi.md`
  - `docs/skills/structure/skill-047-generated-subset-prune.md`
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - `docs/skills/security-testing/skill-062-penetration-testing-tool-workflow.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
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
  - minimally viable auth/base URL posture for direct exploration, or enough context to classify `blocked-auth`
- Optional:
  - subset selection
  - autonomous/no-interrupt execution posture
  - standard-negative coverage intent
  - security coverage intent
  - optional DB comparison intent
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
5. Reuse the stable intake strengths from Skill 033:
   - posture resolution
   - all-vs-subset selection
   - negative/security intent
   - optional DB-comparison follow-on intent
   - deterministic host-asset naming gates

### 6.1.1 API Capability Preflight (Required)
6. Before any write or execution operation, run branch-aware capability preflight via `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.
7. Use the matched Skill 050 profile for the active branch and do not continue with optimistic endpoint assumptions when Skill 050 selects a fallback route.

### 6.2 Experimental Planning and Preanalysis
8. Build the selected operation slice from the OpenAPI plus active project context.
9. If interactive mode is active and scope is still unresolved, present the operation catalog before asking all-vs-subset or subset-selection questions.
10. Build or recompute the transient exploration preanalysis needed for deterministic exploration.
11. Keep that preanalysis transient under `work/`; do not persist exploration-specific structure into `project.yaml`.

### 6.3 Direct Exploration Phase
12. Delegate direct endpoint exploration to Skill 066.
13. Require Skill 066 to produce, per selected operation:
  - a canonical happy-path request when available
  - response-shape and volatility hints
  - candidate standard-negative families
  - a final classification (`usable happy path`, `blocked-auth`, `blocked-input`, `probe-exhausted`, `negative-only-evidence`, or `spec-runtime-mismatch`)
  - a normalized mutation-ready request bundle when successful
14. Treat the exploration ledger as the evidence contract for later authoring and validation planning.
15. Do not reopen a discovery loop through SOAtest runs once the exploration phase has already produced sufficient evidence.

### 6.4 Experimental Approval Boundary (Interactive Required)
16. In interactive mode, build the pre-write approval artifact using `docs/templates/skill-065-approval-artifact-template.md`.
17. The approval artifact must summarize:
  - selected scope and target asset
  - experimental-lane defaults applied
  - exploration-backed happy-path authoring plan per selected operation
  - standard-negative/security/validation intent
  - review items, blockers, or evidence gaps still needing user input
18. Do not begin writes until review items are resolved and the user explicitly approves the experimental plan.
19. In autonomous mode, skip the approval artifact and proceed once blocker questions are resolved.

### 6.5 SOAtest Authoring from Exploration Evidence
20. When the branch is broad new-file authoring, create the host `.tst` through `docs/skills/platform/skill-022-tst-create-from-openapi.md`.
21. If subset scope was selected, prune immediately through `docs/skills/structure/skill-047-generated-subset-prune.md`.
22. Inspect the resulting client family per selected operation.
23. Apply the normalized request bundle through the existing mutation owner:
  - unconstrained / None-mode REST Client updates -> Skill 020
  - constrained same-operation client updates -> Skill 059
24. Preserve normal client-header ownership and read-merge-write safety rules during these writes.

### 6.6 Negative, Security, and Validation Phases
25. Derive standard negatives from the exploration-backed happy-path request and the evidence-backed negative families captured by Skill 066.
26. Keep negative coverage bounded to meaningful request-level mutations.
27. Create security branches by copying the finalized happy-path client and attaching the Penetration Testing Tool through Skill 062.
28. Delegate exploration-backed validation planning and attachment to Skill 067.
29. Keep response-vs-database comparison as an optional later branch rather than part of the mandatory core sequence.

### 6.7 Fallback and Stop Conditions
30. If exploration did not produce a usable happy path for a selected slice:
  - classify that slice explicitly rather than trying to hide the failure behind later SOAtest authoring,
  - stop the slice as blocked or partial, or
  - explicitly hand that slice to Skill 058 when the user wants fallback remediation rather than a pure exploration-backed result.
31. Do not silently reopen the stable generation/remediation loop as though this were the default lane.
32. Stable Skill 058 remains a fallback specialist, not the default next phase.

### 6.8 Focused Verification and Bounded Correction
33. After request configuration and approved validation attachment are complete, run the minimum focused SOAtest verification set:
  - one focused happy-path verification run for each authored slice
  - one focused calibration/verification pass for created standard negatives
  - no default execution of copied security suites
34. Treat that first SOAtest run as verification, not baseline discovery.
35. If verification fails even though direct exploration succeeded:
  - collect the normal results/readback/traffic triad,
  - compare persisted state against the canonical exploration bundle and approved validation plan,
  - allow at most one automatic correction attempt only when the evidence shows a dominant SOAtest-side persistence/configuration defect,
  - otherwise stop as blocked or partial.

### 6.9 Completion Reporting
36. Report the final result using `docs/templates/skill-065-completion-artifact-template.md`.
37. Completion reporting must state:
  - that the lane was experimental and explicit opt-in
  - which slices were delivered, partial, blocked, or deferred
  - that the design baseline was exploration-backed and the SOAtest run was verification-focused

## 7) Validation
- The experimental branch never activates without explicit opt-in.
- The lane remains REST/OpenAPI-only in v1.
- Direct exploration evidence is gathered before SOAtest writes begin.
- The exploration ledger remains transient under `work/`.
- Existing stable leaves remain authoritative for actual SOAtest mutation and execution mechanics.
- Interactive mode does not write before explicit approval of the exploration-backed plan.
- Autonomous mode remains blocker-only once minimally viable inputs are resolved.
- Standard-negative/security/validation phases are derived from exploration-backed happy-path evidence rather than from a separate pre-attachment SOAtest discovery run.
- SOAtest verification remains in scope, but only as a focused persisted-behavior proof.
- Post-authoring correction is bounded to one evidence-backed persistence/configuration repair attempt.

## 8) Failure Modes
- Experimental opt-in is assumed from vague wording and the branch is activated incorrectly.
- SOAP or non-OpenAPI authoring is forced into the experimental lane.
- Direct endpoint exploration is treated as interchangeable with stable execution-backed evidence outside this explicit lane.
- Exploration evidence is not captured clearly enough to drive later authoring and validation.
- The lane invents a parallel mutation stack instead of delegating to existing stable leaves.
- Skill 058 fallback is used as the default next phase rather than as an explicit fallback.
- A failed focused verification run reopens an unbounded rediscovery loop.
- Security branches are pulled into interim verification runs even though no explicit security execution was requested.

## 9) Safety / Rollback
- Treat the experimental lane as write-capable and side-effect-capable because direct exploration may include bounded state-changing CRUD probes.
- Keep all exploration artifacts and ledgers transient under `work/`.
- Delegate object-level rollback to the downstream stable leaves used for each mutation surface.
- Stop rather than guessing when auth, business data, schema-source confirmation, or runtime evidence remain insufficient.

## 10) Reuse Notes
- This is the top-level explicit opt-in owner for the experimental lane.
- Use stable Skill 033 when experimental opt-in is absent.
- Use Skill 066 for the direct exploration rules and evidence contract.
- Use Skill 067 for exploration-backed validation planning and attachment.
- Keep stable Skills 033, 057, and 058 unchanged as the default operator-facing branch outside this lane.
