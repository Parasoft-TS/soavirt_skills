# Skill 067: Experimental Validation-Enrichment Orchestration (Composite)

## 1) Skill Name
Exploration-backed validation-enrichment orchestration for the explicit opt-in experimental live-exploration lane.

## 2) Objective
Convert exploration-backed response evidence into an approved validation-enrichment plan for authored SOAtest tests without requiring a separate pre-attachment SOAtest baseline run, while preserving the stable approval model, schema-source confirmation rules, producer-output chaining rules, and focused post-attachment verification requirements.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting validation scope and approval.

## 3) Scope
- In scope:
  - validation planning for tests authored through the experimental lane
  - exploration-backed happy-path validation-bundle design
  - standard-negative validation overlay for exploration-backed negatives
  - no-tool exceptions for empty or HTML responses
  - explicit schema-source confirmation before validator attachment
  - deterministic handoff to existing validator/assertor/diff leaves
  - focused post-attachment SOAtest verification
  - bounded repair for validation-tool persistence/configuration defects that preserve approved intent
- Out of scope:
  - stable default validation-enrichment routing when exploration-backed evidence is absent
  - replacing validator/assertor/diff lifecycle leaves
  - inventing schema/service-definition sources
  - silently weakening or substituting approved validation coverage
  - security-branch validator/assertor/diff attachment by default

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- Additive:
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - `docs/skills/validation/skill-016-xml-assertor-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`
  - `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`

## 4) Inputs
- Required:
  - canonical exploration dossier for the selected authored slice
  - canonical happy-path request/response pair chosen by the exploration phase
  - current authored producer identity in SOAtest
- Optional:
  - reconciliation response evidence from the exploration phase
  - approved standard-negative/security/DB comparison scope
  - schema/service-definition candidates from intake, bootstrap, or generated asset context
  - explicit user-specified checks

## 5) Preconditions
- The branch is inside the explicit opt-in experimental lane.
- The selected slice already has exploration-backed authoring evidence.
- The authored producer exists in SOAtest and is resolvable.
- Validator creation does not proceed until schema/service-definition inputs are explicitly provided or explicitly confirmed.

## 6) Procedure
### 6.1 Evidence Intake and Legitimacy Gate
1. Accept exploration evidence as the primary design input only when it represents the same selected operation intent that will be validated in SOAtest.
2. Require the exploration evidence to satisfy all of the following:
  - the happy-path slice ended in `usable happy path`, or the negative slice carries explicitly accepted evidence-backed negative behavior
  - payload/media-type classification is based on the observed semantic response body first
  - the evidence comes from the canonical request/response pair chosen for authoring
  - the evidence is not merely a pre-remediation or underconfigured output symptom
3. If legitimacy remains weak or ambiguous, stop the slice rather than inventing a validation bundle from low-confidence evidence.

### 6.2 Approval Boundary
4. The experimental validation card keeps the same approval model as stable Skill 057:
  - inspect evidence
  - propose the validation bundle
  - require explicit user approval before validation writes
5. Approval to explore or author happy-path requests is not approval to attach validators/assertors/diff tools.
6. Once the user approves validation coverage for a target:
  - do not silently omit approved coverage
  - do not silently downgrade validator coverage to content-only tooling
  - do not silently replace an approved assertor branch with Diff Tool
  - do not reinterpret approved field-level assertions as full-response diffing

### 6.3 Happy-Path Bundle Selection
7. Choose validation family from the observed semantic response payload/body captured during exploration:
  - observed JSON response -> JSON Validator plus JSON Assertor or Diff Tool JSON mode
  - observed XML response -> XML Validator plus XML Assertor or Diff Tool XML mode
  - observed plain text or other non-JSON/XML response -> Diff Tool only in the mode implied by the observed payload/body
8. Apply the stable content-validation heuristic:
  - mostly static-looking payload -> prefer Diff Tool
  - obviously volatile payload -> prefer Assertor
  - when uncertain, choose one approved content-validation family without repeated exploratory reruns in v1
9. If the observed response payload/body is empty or HTML, treat the target as a no-tool exception and keep the REST Client expected response code as the validation outcome.

### 6.4 Schema-Source Confirmation
10. Exploration evidence does not authorize schema/source assumptions by itself.
11. For validator branches:
  - schema/service-definition sources must still come from explicit user input or clearly discoverable candidates presented back for confirmation
  - environment-variable-backed candidates must retain both variable form and resolved value during confirmation
  - do not invent synthetic schemas or placeholder definition locations
12. Keep stable validator strictness:
  - OpenAPI-backed JSON validation keeps explicit message mapping with template paths rather than concretized runtime values
  - XML validation keeps explicit schema-location behavior and referenced-schema requirements
13. If schema/service-definition source is missing or ambiguous, block validator attachment for that target and return for user decision instead of silently downgrading the whole bundle.

### 6.5 Standard-Negative Overlay
14. Standard negatives remain a policy overlay rather than a separate intake branch.
15. For selected exploration-backed standard negatives:
  - do not attach JSON/XML validators by default
  - choose content-validation family from the observed negative payload/body, not from the happy-path media type
  - if the negative response body is empty or HTML, keep expected response code as the only validation
  - prefer Diff Tool for simple static-looking error payloads
  - prefer Assertor when targeted error-field checks are more meaningful
16. If the authored negative still relies on default expected-response-code behavior, require expected-code correction/calibration before declaring validation complete.

### 6.6 Security and DB Exceptions
17. Security branches created through the Penetration Testing Tool workflow remain exempt from validator/assertor/diff attachment by default.
18. Report those branches as intentionally handled by the penetration-testing workflow.
19. If the approved plan includes response-vs-database comparison, keep it as a separate optional branch routed through the existing DB-comparison workflow family after the primary API validation bundle is defined.

### 6.7 Orchestration Mapping
20. Attach validation through the existing stable leaves only:
  - JSON Assertor -> Skill 010
  - XML Assertor -> Skill 016
  - JSON Validator -> Skill 029
  - XML Validator -> Skill 030
  - Diff Tool -> Skill 031
21. Preserve their existing constraints:
  - producer-output-only chaining
  - idempotent upsert where defined
  - read-merge-write for updates
  - family-specific fail-closed media-type gates

### 6.8 Confirmation and Execution
22. Build a human-readable validation plan summary that includes:
  - selected targets
  - evidence source as exploration-backed
  - readiness status
  - per-target proposed validation family
  - no-tool exceptions
  - schema-source status
  - negative/security/DB exceptions
23. Require explicit user confirmation of this validation plan before validation writes.
24. After approval, attach/configure validation tools in the approved sequence and verify readback.

### 6.9 Focused Verification After Attachment
25. After validation tools are attached, run one focused SOAtest verification execution for the selected happy-path slice and inspect the normal run-results-traffic triad.
26. The goal of that run is narrow:
  - confirm the authored producer still behaves as expected inside SOAtest
  - confirm the chained validation tools are attached to the intended semantic output
  - confirm family selection remains consistent with the authored runtime payload
27. Do not insert a separate pre-attachment discovery run when exploration evidence already supplied a trustworthy design baseline.

### 6.10 Validation-Side Bounded Repair
28. Apply the bounded authoring-correction model to validation-tool persistence defects only when the repair preserves approved validation intent.
29. Eligible one-pass repair examples include:
  - approved tool attached to the wrong output-provider parent
  - approved tool missing after authoring because persistence failed
  - approved validator persisted with configuration contradicting the already-approved source or mapping
  - approved Diff Tool persisted in the wrong mode relative to the authored runtime payload
30. Non-eligible repairs include:
  - switching from validator to assertor or diff because validator setup is harder
  - weakening assertor/diff strictness after mismatch without explicit user confirmation
  - substituting a new schema/service-definition source that was not user-provided or user-confirmed
  - dropping approved coverage after approval

## 7) Validation
- Exploration-backed evidence replaces only the stable baseline run, not the approval model.
- Validation-tool attachment does not occur before explicit plan confirmation.
- Family selection is driven by the observed exploration-backed response body, not only by producer class or headers.
- Schema-validator strictness remains unchanged from the stable leaves.
- Empty or HTML response bodies are no-tool exceptions that rely on the REST Client expected response code alone.
- Standard negatives do not receive JSON/XML validators by default.
- Security branches remain validator/assertor/diff-free by policy unless a future explicit branch says otherwise.
- Post-attachment SOAtest verification remains mandatory.
- Validation-side bounded repair remains one-pass and preserves approved intent.

## 8) Failure Modes
- Exploration evidence is treated as sufficient even though it came from an abandoned alternate request.
- A validation bundle is proposed from response headers rather than the observed body.
- Validator/assertor/diff tools are attached even though the observed response body is empty or HTML and expected response code alone should have remained the validation.
- Validator coverage is silently downgraded when source confirmation is missing.
- Negative tests inherit the happy-path validator bundle by default.
- Security branches receive validator/assertor/diff tooling even though the policy excludes it.
- Leaf skills silently substitute a different family after approval.
- Post-attachment verification is treated as a discovery run and used to redesign the bundle repeatedly.

## 9) Safety / Rollback
- Avoid speculative validation writes before explicit approval.
- Delegate tool-level rollback to the existing validation leaves.
- Stop and report blocked/partial states when approved coverage cannot be delivered without a fresh user decision.

## 10) Reuse Notes
- This card is the experimental validation owner expected to be called from Skill 065.
- Use stable Skill 057 for the default execution-backed validation lane.
- Keep this card additive; it changes the baseline evidence source, not the validator/assertor/diff lifecycle owners.
