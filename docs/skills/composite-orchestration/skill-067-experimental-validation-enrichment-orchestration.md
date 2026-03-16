# Skill 067: Experimental Validation-Enrichment Orchestration (Composite)
## 1) Skill Name
Exploration-backed validation-enrichment orchestration for the explicit opt-in experimental live-exploration lane.
## 2) Objective
Convert exploration-backed response evidence into a posture-aware validation-enrichment workflow for authored SOAtest tests without requiring a separate pre-attachment SOAtest baseline run, while preserving stable validator/assertor/diff lifecycle mechanics, schema-source confirmation rules, producer-output chaining rules, and focused post-attachment verification requirements.
Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting validation scope and any genuinely unresolved review items.
## 3) Scope
- In scope:
  - validation planning for tests authored through the experimental lane
  - exploration-backed happy-path validation-bundle design
  - standard-negative validation overlay for exploration-backed negatives
  - no-tool exceptions for empty or HTML responses
  - explicit schema-source confirmation before validator attachment
  - deterministic handoff to existing validator/assertor/diff and databank leaves
  - databank reuse/update rules for enrichment-only extracts
  - focused post-attachment SOAtest verification, normally through Skill 074 structural isolation
  - bounded repair for validation-tool persistence/configuration defects that preserve approved intent
  - posture inheritance from Skill 065, including autonomous no-separate-approval follow-through unless a true blocker remains
- Out of scope:
  - stable default validation-enrichment routing when exploration-backed evidence is absent
  - replacing validator/assertor/diff lifecycle leaves
  - replacing databank lifecycle leaves
  - inventing schema/service-definition sources
  - silently weakening or substituting approved validation coverage
  - security-branch validator/assertor/diff attachment by default
## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/structure/skill-074-set-disabled-state.md`
- Additive:
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - `docs/skills/validation/skill-016-xml-assertor-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`
  - `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
## 4) Inputs
- Required:
  - one Skill 065 validation handoff packet for the selected authored slice, including:
    - cluster/slice identity and authored scenario path
    - authored producer identity and producer role
    - canonical exploration evidence reference and canonical happy-path request/response pair
    - exploration-backed payload/media-type classification
    - candidate expected-content basis and volatility hints
    - schema/service-definition candidates with provenance
    - validator message-mapping basis for validator-capable slices
    - available databanked bindings from prior steps
    - existing producer-local databank presence when one already exists
    - candidate cross-step semantic comparisons
    - recommended focused verification scope
    - per-slice readiness/status suggestion
- Optional:
  - approved standard-negative/security/DB comparison scope
  - explicit user-specified checks
## 5) Preconditions
- The branch is inside the explicit opt-in experimental lane.
- The selected slice already has exploration-backed authoring evidence.
- The authored producer exists in SOAtest and is resolvable.
- Validator creation does not proceed until schema/service-definition inputs are explicitly provided or explicitly confirmed.
## 6) Procedure
### 6.1 Evidence Intake and Legitimacy Gate
1. Accept the Skill 065 handoff packet as the caller-owned synchronization contract for this slice.
2. Treat the Skill 066 ledger as advisory input to that packet rather than as a rigid validation contract.
3. Accept exploration evidence as the primary design input only when it represents the same selected authored slice intent that will be validated in SOAtest.
4. Require the exploration evidence to satisfy all of the following:
  - the happy-path slice ended in `usable happy path`, or the negative slice carries explicitly accepted evidence-backed negative behavior
  - payload/media-type classification is based on the observed semantic response body first
  - the evidence comes from the canonical request/response pair chosen for authoring
  - the evidence is not merely a pre-remediation or underconfigured output symptom
5. If legitimacy remains weak or ambiguous, stop the slice rather than inventing a validation bundle from low-confidence evidence.
### 6.2 Posture and Approval Boundary
6. Preserve Skill 065 posture exactly.
7. In interactive runs:
  - use the Skill 065 approval/review flow as the user-facing approval surface for validation intent and unresolved review items
  - do not create a second orphan validation approval artifact
  - surface only genuinely unresolved questions such as schema/source confirmation or ambiguous family choice
8. In autonomous runs:
  - do not require a separate approval artifact by default
  - proceed from the Skill 065 handoff packet unless a true blocker remains
  - return blockers in one aggregated set when batching is possible
9. Approval to explore or author happy-path requests is not by itself approval to attach validators/assertors/diff tools; the approved Skill 065 handoff and posture-specific review flow supply that authority in this lane.
10. Once validation coverage is approved for a target:
  - do not silently omit approved coverage
  - do not silently downgrade validator coverage to content-only tooling
  - do not silently replace an approved assertor branch with Diff Tool
  - do not reinterpret approved field-level assertions as full-response diffing
### 6.3 Happy-Path Bundle Selection
11. Treat the exploration-backed payload/media-type classification in the Skill 065 handoff packet as authoritative for validation-family selection.
12. Choose validation family from that classification and the candidate expected-content basis:
  - JSON response -> JSON Validator plus JSON Assertor or Diff Tool JSON mode
  - XML response -> XML Validator plus XML Assertor or Diff Tool XML mode
  - plain text or other non-JSON/XML response -> Diff Tool only in the mode implied by the observed payload/body
12a. For non-empty JSON/XML happy-path responses, the default enrichment target is a two-part bundle: one schema validator plus one content-validation tool (`Assertor` or `Diff Tool`) justified by the volatility heuristic. Do not stop after attaching only the validator unless the slice later lands in `no-tool exception`, `blocked`, or `delivered-valid-finding` with the remaining gap reported explicitly.
13. Use the handoff packet's candidate expected-content basis as authoritative starting evidence for content-validation design.
14. Apply the stable content-validation heuristic:
  - mostly static-looking payload whose approved intent is stable whole-payload equality -> prefer Diff Tool
  - obviously volatile payload -> prefer Assertor
  - when the expected semantic content is inferred from earlier business responses or from mutable shared/business state likely to drift across runs (for example balances, positions, transaction collections, approval echoes, or mutable profile fields), prefer targeted Assertor checks or explicit cross-step semantic comparisons over whole-response Diff even if the payload looks structurally stable
  - when uncertain, choose one approved content-validation family without repeated exploratory reruns in v1
15. If the observed response payload/body is empty or HTML, treat the target as a no-tool exception and keep the REST Client expected response code as the validation outcome.
15a. Validation/source ordering rule:
  - payload/body family selection comes before schema-source confirmation
  - validator-source confirmation governs validator attachment only; it does not decide the content-validation family
  - if validator source is missing for a JSON/XML slice, keep the correct Assertor-or-Diff decision and report validator blockage rather than reshaping content validation to fit source availability
### 6.4 Schema-Source Confirmation
16. Exploration evidence authorizes family selection and candidate expected content, but it does not by itself authorize schema/source assumptions.
17. For validator branches:
  - schema/service-definition sources must still come from explicit user input or clearly discoverable candidates presented back for confirmation in interactive mode
  - autonomous mode may proceed only when the handoff packet already carries one unambiguous confirmed source or one source already approved upstream for this branch
  - environment-variable-backed candidates must retain both variable form and resolved value during confirmation
  - do not invent synthetic schemas or placeholder definition locations
18. Keep stable validator strictness:
  - OpenAPI-backed JSON validation keeps explicit message mapping with template paths rather than concretized runtime values
  - XML validation keeps explicit schema-location behavior and referenced-schema requirements
18a. Deterministic validator message-mapping rule:
  - when a validator bundle uses OpenAPI mapping, derive `message.path`, `message.method`, and `message.responseCode` from the Skill 065 handoff packet in this order:
    1. exact authored producer operation identity or same-slice lineage
    2. confirmed OpenAPI path template and method for that same operation
    3. expected response-code basis from the authored slice or approved calibration context
  - treat observed concrete request URLs, paraphrased resource names, or semantic guesses as confirmation aids only, not as authority to invent the mapping
  - if those inputs disagree or do not converge on one exact OpenAPI operation template, block the validator branch rather than guessing
19. If schema/service-definition source is missing or ambiguous:
  - block validator attachment for that target only
  - continue with assertor/diff when justified
  - report the slice as `validator-blocked-but-assertor/diff-delivered` when appropriate rather than blocking the whole validation slice
### 6.5 Databank Reuse and Validation-Only Extracts
20. When enrichment needs additional extracted values for an authored producer already configured by Skill 065, inspect that producer's existing databank situation first.
21. If a suitable producer-local databank already exists under the producer:
  - update that existing databank through the appropriate leaf (`Skill 028` for JSON, `Skill 019` for XML)
  - add the new validation-only extraction entries into that databank instead of creating a sibling databank
22. Only when no suitable databank exists should Skill 067 create a new producer-local databank.
23. Treat databanks created by Skill 065 as the preferred host for later enrichment-only extracts unless doing so would corrupt already-approved execution-critical mappings.
### 6.6 Cross-Step Semantic Comparison Rule
24. Skill 067 may auto-add a cross-step semantic validation comparison only when all of these hold:
  - the source value comes from the same authored cluster/scenario
  - the provenance is explicit in the Skill 065 handoff packet
  - the semantic relationship is clear, for example identity continuity, echoed field, foreign-key/reference relationship, or create/read/reconcile continuity
  - the value is not obviously volatile
  - there is not an equally plausible competing source binding
  - the comparison adds semantic coverage beyond request plumbing
  - the focused verification scope will execute the prerequisite producer so the value exists at runtime
25. If any condition fails, treat the comparison as a candidate rather than auto-attaching it.
### 6.7 Standard-Negative Overlay
26. Standard negatives remain a policy overlay rather than a separate intake branch.
27. For selected exploration-backed standard negatives:
  - do not attach JSON/XML validators by default
  - choose content-validation family from the observed negative payload/body, not from the happy-path media type
  - if the negative response body is empty or HTML, keep expected response code as the only validation
  - prefer Diff Tool for simple static-looking error payloads
  - prefer Assertor when targeted error-field checks are more meaningful
28. If the authored negative still relies on default expected-response-code behavior, require expected-code correction/calibration before declaring validation complete.
### 6.8 Security and DB Exceptions
29. Security branches created through the Penetration Testing Tool workflow remain exempt from validator/assertor/diff attachment by default.
30. Report those branches as intentionally handled by the penetration-testing workflow.
31. If the approved plan includes response-vs-database comparison, keep it as a separate optional branch routed through the existing DB-comparison workflow family after the primary API validation bundle is defined.
### 6.9 Orchestration Mapping
32. Attach validation through the existing stable leaves only:
  - JSON Data Bank -> Skill 028
  - XML Data Bank -> Skill 019
  - JSON Assertor -> Skill 010
  - XML Assertor -> Skill 016
  - JSON Validator -> Skill 029
  - XML Validator -> Skill 030
  - Diff Tool -> Skill 031
33. Preserve their existing tool-level constraints:
  - producer-output-only chaining
  - idempotent upsert where defined
  - read-merge-write for updates
  - family-specific fail-closed media-type gates
34. When called from Skill 067, those leaves may accept upstream resolution of payload/media-type classification, approved validation family, candidate expected-content basis, and target producer/output selection instead of rediscovering those facts through a separate stable-lane family-selection loop.
### 6.10 Confirmation, Execution, and Completion State
35. Build a per-slice validation plan summary that includes:
  - selected target
  - evidence source as exploration-backed
  - readiness status
  - proposed validation family
  - no-tool exceptions
  - schema-source status
  - validator message-mapping status
  - candidate cross-step comparisons
  - databank reuse/new-create decision
  - negative/security/DB exceptions
36. After approval or autonomous continuation under the inherited posture, attach/configure validation tools in the approved sequence and verify readback.
37. After attachment, run one focused SOAtest verification execution for the selected authored slice and inspect the normal run-results-traffic triad.
38. Prefer the smallest structural verification scope that matches the approved slice:
  - one `Baseline Snapshot` or `Safe Reference Reads` subtree for read-only happy-path validation
  - one `Lifecycle Flow` scenario sub-suite for lifecycle happy-path validation
  - one negative bundle for selected negative-overlay validation
39. When the authored tree supports structural isolation, prefer Skill 074 disable-run-restore sandboxing before falling back to `soatestOptions.testNames`.
40. The goal of that run is narrow:
  - confirm the authored producer still behaves as expected inside SOAtest
  - confirm the chained validation tools are attached to the intended semantic output
  - confirm family selection remains consistent with the authored runtime payload
40a. If post-attachment verification fails, classify the outcome before changing tools:
   - persistence/configuration defect in the authored validation branch
   - genuine service/contract/content finding with the validation branch configured correctly
40b. For validator failures, treat the finding as genuine when the configured parent/output selection, payload family, schema/service-definition source, message mapping, and expected response-code context are correct and the observed payload still violates the confirmed contract/schema.
40c. When the failure is a genuine finding, keep the validator/assertor/diff branch intact, preserve the failing result as evidence, and report the slice as `delivered-valid-finding` rather than deleting or weakening the tooling to force a pass.
41. Do not insert a separate pre-attachment discovery run when exploration evidence already supplied a trustworthy design baseline.
42. For each happy-path slice, end in exactly one state:
  - `delivered`
  - `delivered-valid-finding`
  - `no-tool exception`
  - `validator-blocked-but-assertor/diff-delivered`
  - `blocked`
### 6.11 Validation-Side Bounded Repair
43. Apply the bounded authoring-correction model to validation-tool persistence defects only when the repair preserves approved validation intent.
44. Eligible one-pass repair examples include:
  - approved tool attached to the wrong output-provider parent
  - approved tool missing after authoring because persistence failed
  - approved validator persisted with configuration contradicting the already-approved source or mapping
  - approved Diff Tool persisted in the wrong mode relative to the authored runtime payload
  - approved validation-only extraction omitted from the reused databank even though the leaf update succeeded incompletely
45. Non-eligible repairs include:
  - switching from validator to assertor or diff because validator setup is harder
  - weakening assertor/diff strictness after mismatch without explicit user confirmation
  - substituting a new schema/service-definition source that was not user-provided or user-confirmed
  - dropping approved coverage after approval
  - deleting, disabling, or downgrading a correctly configured validator merely because it surfaced a genuine schema/contract defect
## 7) Validation
- Exploration-backed evidence replaces the stable baseline-discovery run for this lane.
- Skill 067 consumes a caller-owned Skill 065 handoff packet rather than rediscovering the full validation context from scratch.
- Validation follow-through preserves Skill 065 posture; autonomous runs do not require a separate validation approval artifact by default.
- For non-empty JSON/XML happy-path slices, delivered enrichment normally includes both schema validation and one content-validation tool rather than validator-only attachment.
- Family selection is driven by the exploration-backed payload/body classification in the handoff packet, not only by producer class or headers.
- Candidate expected content may rely on exploration evidence as baseline authority.
- Empty or HTML response bodies are no-tool exceptions that rely on the REST Client expected response code alone.
- Validator schema/source ambiguity blocks validator attachment only; assertor/diff may still proceed when justified.
- Validator message mapping is derived deterministically from the authored producer identity plus the confirmed OpenAPI operation template rather than from semantic guesswork or concretized runtime URLs.
- Standard negatives do not receive JSON/XML validators by default.
- Security branches remain validator/assertor/diff-free by policy unless a future explicit branch says otherwise.
- Validation-only extractions reuse the existing producer-local databank before creating a sibling databank.
- Cross-step semantic validations auto-attach only when the relationship is strong enough under Section 6.6.
- Post-attachment SOAtest verification remains mandatory and should prefer Skill 074 structural isolation over coarse `testNames` filtering when the authored tree supports it.
- Correctly configured validator/assertor/diff failures that reveal genuine service defects are valid findings and should be reported as `delivered-valid-finding` rather than auto-removed.
- Validation-side bounded repair remains one-pass and preserves approved intent.
## 8) Failure Modes
- Skill 067 ignores the Skill 065 handoff packet and re-derives validation context from scratch.
- Exploration evidence is treated as sufficient even though it came from an abandoned alternate request.
- A validation bundle is proposed from response headers rather than the observed body.
- Validator/assertor/diff tools are attached even though the observed response body is empty or HTML and expected response code alone should have remained the validation.
- Validator coverage blocks the entire slice even though assertor/diff could have proceeded.
- Validator message mapping is guessed from paraphrased semantics or concrete runtime URLs instead of being derived from the exact authored operation and confirmed OpenAPI template.
- A second sibling databank is created under a producer even though an existing producer-local databank could have been updated instead.
- Cross-step comparisons are auto-added even though provenance, semantic relationship, volatility, or verification scope support is too weak.
- Negative tests inherit the happy-path validator bundle by default.
- Happy-path validation is treated as complete after attaching only a validator even though a content-validation tool was still required for a non-empty JSON/XML slice.
- Security branches receive validator/assertor/diff tooling even though the policy excludes it.
- A genuine validator finding is mistaken for misconfiguration and the correctly configured validator is removed or weakened just to make the run pass.
- Leaf skills silently substitute a different family after approval.
- Post-attachment verification is treated as a discovery run and used to redesign the bundle repeatedly.
- Post-attachment verification falls back to coarse `testNames` filtering even though structural isolation through Skill 074 was practical for the selected authored slice.
## 9) Safety / Rollback
- Avoid speculative validation writes before posture-appropriate approval or continuation authority exists.
- Delegate tool-level rollback to the existing databank/validator/assertor/diff leaves.
- Stop and report blocked/partial states when approved coverage cannot be delivered without a fresh user decision.
## 10) Reuse Notes
- This card is the experimental validation owner expected to be called from Skill 065.
- Use stable Skill 057 for the default execution-backed validation lane.
- Keep this card additive; it changes the evidence and orchestration contract, not the underlying validator/assertor/diff lifecycle owners.
