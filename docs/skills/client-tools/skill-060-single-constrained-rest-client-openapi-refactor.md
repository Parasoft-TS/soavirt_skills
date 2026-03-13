# Skill 060: Single Constrained REST Client OpenAPI Refactor

## 1) Skill Name
Refactor one constrained REST Client subtree from one confirmed OpenAPI contract to another.

## 2) Objective
Refactor exactly one source-spec-bound constrained REST Client subtree from a confirmed source OpenAPI contract to a confirmed target OpenAPI contract, including one-client operation/parameter/request-body migration reasoning and supported downstream chained-tool repair, while returning structured per-client analysis/write results and enforcing client-slice structural validation plus rollback.

This card is the lower-layer reusable leaf beneath broader bulk orchestration such as `Change Advisor`. It is not a multi-client inventory owner and it does not own grouped user review across multiple targets.

## 3) Scope
- In scope:
  - one constrained REST Client subtree only
  - one confirmed source OpenAPI -> one confirmed target OpenAPI migration slice
  - two modes:
    - `analysis`
    - `write`
  - operation mapping for the selected constrained REST Client
  - path/query parameter mapping for the selected client
  - request-body field mapping for the selected client
  - supported downstream chained-tool repair intent and application for:
    - JSON Validator
    - JSON Data Bank
    - JSON Assertor
    - Diff Tool
  - structured per-client result emission for orchestration use
  - client-slice structural verification and rollback
- Out of scope:
  - multi-client inventory or bulk orchestration ownership
  - grouped user review ownership across multiple targets
  - copied-file naming or whole-run write sequencing
  - unconstrained REST migration
  - SOAP/WSDL migration
  - XML-family chained-tool refactoring
  - family substitution beyond the approved v1 repair rules
  - multi-spec or multi-target migrations in one invocation

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
- Additive:
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`

## 4) Inputs
- Required:
  - mode:
    - `analysis`
    - `write`
  - stable client reference:
    - source `.tst` path context (suite/scenario/test path)
    - target client identity within that path
  - source client YAML/config slice
  - immediate supported chained-tool descendants for that client
  - confirmed source OpenAPI subset needed for that client
  - confirmed target OpenAPI subset needed for that client
  - normalized one-client comparison context:
    - source operation definition
    - target operation candidates or resolved target operation
    - relevant source/target request schema subset
    - relevant source/target response schema subset for supported downstream tools
  - run-policy constraints for this branch:
    - exact source-spec rewrite only
  - approved base URL policy for this branch:
    - keep current/source (default when no override was approved)
    - rewrite exact-match source base URL values to the approved replacement value
- Conditionally required for `write` mode:
  - copied target `.tst` context
  - exact client-slice location inside the copied `.tst`
  - resolved decisions for that client only
  - approved chained-tool repair contracts for any non-autonomous downstream changes
- Optional:
  - same-`.tst` supporting evidence for value sourcing or selector meaning
  - orchestration-provided evidence anchors for source/target diff rationale
  - precomputed autonomous classifications from an earlier analysis pass when `write` mode is invoked separately from the original `analysis` pass

## 4.0) Evidence Boundary
- This leaf should reason from:
  - the downloaded/source-of-truth `.tst` YAML slice for the target client subtree, and
  - normalized source/target OpenAPI structures for that same client slice.
- Do not mix ad hoc post-download API rediscovery of constrained-client internals into the refactor-analysis evidence model.
- Same-`.tst` supporting evidence may help with value sourcing or selector meaning, but it does not override stronger client YAML or normalized OpenAPI evidence.

## 4.1) Result Contract
This leaf must return structured per-client results rather than only prose.

For `analysis` mode, return:
- client identity
- operation mapping result including:
  - chosen target operation when known
  - confidence classification
  - evidence summary
- parameter mapping result including:
  - kept/dropped/omitted-optional decisions
  - proposal items
  - blocked items
- request-body mapping result including:
  - autonomous carry-forward decisions
  - proposal items
  - blocked items
  - generated candidate values and rationale for target-only fields when one schema-conformant proposal exists
  - explicit omission proposals when omission is the candidate strategy
  - structured unresolved body-review items, each containing:
    - target field path
    - source field/path basis when one exists
    - change type (`carry-forward`, `rename`, `nesting-move`, `target-only-field`, `omission`, or `type-conversion`)
    - candidate value when one is proposed
    - omission proposal when applicable
    - rationale
    - confidence
    - required-vs-optional classification
- supported chained-tool repair intents, one record per supported downstream tool, including:
  - tool identity
  - tool family
  - repair kind
  - confidence classification
  - old selector/binding/baseline anchor when relevant
  - new selector/binding/baseline proposal when known
- per-domain decision-state record:
  - `operationState`
  - `parameterState`
  - `requestBodyState`
  - `chainedToolState`
  - use `NOT_APPLICABLE` only when the domain truly does not apply
- overall client decision-state (`AUTO`, `REVIEW`, or `BLOCKED`)
  - compute this as the worst applicable per-domain state
  - for body-bearing operations, `requestBodyState` is mandatory; missing, unresolved, or `NOT_APPLICABLE` prevents overall `AUTO`
- client-local unresolved decision items including mapping layer, decision prompt, proposal text when one exists, rationale, and dependency references
- predicted write plan in dependency order

For `write` mode, return:
- client identity
- applied constrained REST Client edits in write order
- applied chained-tool edits in write order
- skipped or deferred edits
- client-slice structural verification result
- final client write state (`applied`, `partial`, `blocked`, or `rolled back`)
- rollback notes and focused follow-up notes when needed

## 4.2) Domain-Completeness Gate
- Analysis must produce an explicit decision-state for every applicable domain:
  - operation
  - parameters
  - request body
  - chained tools
- Exact path+method continuity never short-circuits later applicable domains.
- `requestBodyState=NOT_APPLICABLE` is valid only when both the source and target operations have no request body.
- For any client where the source or target operation has a request body, an explicit request-body mapping result and `requestBodyState` are mandatory.
- If an applicable domain result is missing, treat the client as incomplete and fail closed (`BLOCKED`) rather than inferring `AUTO`.

## 5) Preconditions
- The target client is already resolved and is confirmed to be an existing constrained REST Client within Skill 059's validated boundary.
- The client's source-spec binding exactly matches the confirmed source OpenAPI anchor for the current branch.
- The source and target OpenAPI inputs are already resolved enough to build a one-client source-vs-target comparison.
- The leaf's evidence inputs already reflect the source-of-truth YAML slice plus normalized source/target OpenAPI structures for this client.
- `write` mode must not begin while client-local unresolved decisions remain.
- `write` mode assumes the copied target `.tst` already exists and the original source `.tst` remains untouched.
- Supported downstream tool families are limited to the v1 matrix in this card.
- If the client's required downstream repair would depend on unsupported XML-family or broader family-conversion behavior, this leaf must classify the client as `blocked` or `partial` rather than improvising a broader migration.

## 6) Procedure
0. Respect orchestration ownership boundaries:
   - do not perform broad run-level inventory here
   - do not assign global review ids here
   - do not ask the user grouped-review questions here
   - do not own copied-file naming or whole-run sequencing here
1. Resolve mode:
   - `analysis`
   - `write`
2. Confirm the client remains within the validated constrained boundary for this leaf:
   - existing constrained REST Client
   - source operation identity expressed as OpenAPI path + HTTP method
   - payload/shape branch remains within what the downstream constrained lifecycle mechanics can safely apply
3. Build the one-client evidence bundle:
   - source client identity and path context
   - source-spec binding evidence
   - source operation definition
   - target operation candidates or resolved target operation
   - immediate downstream supported tool descendants
3.1. Maintain source-of-truth discipline:
   - treat the downloaded `.tst` YAML slice plus normalized source/target OpenAPI structures as the authoritative analysis inputs
   - do not broaden the evidence base with ad hoc runtime rediscovery of constrained internals once this bundle is established

### 6.1) Analysis Mode
4. Analyze operation mapping first:
   - compare source operation and target candidates using path + HTTP method as the primary key
   - generate candidates in descending strength:
     - exact same path + HTTP method in target
     - same HTTP method + same normalized path skeleton with only path-parameter token-name differences
     - one strong replacement candidate supported by contract overlap
   - treat `operationId`, summary/description text, and schema similarity as supporting evidence only; they do not override stronger path+method evidence
   - classify operation outcome as:
     - `safe-autonomous` when exact same path+method exists, or one unique normalized-path-skeleton successor exists with no competing candidate
     - `proposal-plus-approval` when one plausible renamed/moved successor exists but semantic judgment is still involved
     - `blocked` when there is no plausible target, multiple plausible targets, split/merge evidence, or the target branch falls outside the validated constrained-client lifecycle boundary
4.1. Apply the domain-completeness guard before moving on:
   - do not let a `safe-autonomous` operation result short-circuit later applicable domains
   - if the source or target operation has a request body, the overall client state remains unresolved until Step 6 produces an explicit request-body result
5. Analyze path/query parameter mapping only after operation context is known:
   - compare source vs target parameter surfaces separately from request-body fields
   - use source constrained path/query YAML, mapped target operation parameter definitions, and same-`.tst` value provenance only when needed
   - treat query-parameter mapping as one logical decision even when multiple persisted YAML mirrors must later be updated together
   - apply default handling rules:
     - preserved same-role parameters with compatible type/shape and still-valid values are candidates for carry-forward
     - source-only parameters absent from target may be dropped by default
     - new optional target parameters may be omitted by default
     - path-token rename may be autonomous only when uniquely implied by a `safe-autonomous` operation mapping
   - classify parameter changes as:
     - `safe-autonomous` when meaning is mechanically preserved and value carry-forward remains valid
     - `proposal-plus-approval` when one plausible rename/location/value mapping exists but semantic confirmation is still required
     - `blocked` when multiple plausible targets compete, a new required target parameter has no defensible value source, or type/shape drift has no clear carry-forward rule
6. Analyze request-body mapping only after operation context is known:
   - compare source vs target request-body field structure, requiredness, and nesting only after the target operation context is sufficiently resolved
   - exact path+method continuity proves only operation continuity; it does not imply request-body compatibility
   - distinguish mechanical carry-forward from semantic inference:
     - same-name, same-role, type-compatible fields with still-valid values are candidates for carry-forward
     - source-only fields absent from target may be omitted by default unless removal changes request meaning in a way this leaf cannot justify
     - target-only fields with no source equivalent always require explicit analysis; they are never `safe-autonomous` merely because they are optional
   - when a target-only field has no direct source mapping:
     - if exactly one schema-conformant candidate can be synthesized from contract defaults, examples, enums, format hints, or adjacent same-object values, return `proposal-plus-approval` with that candidate and rationale
     - if omission is a defensible strategy, include it as an explicit proposal rather than silently taking it
     - if the field is required and no defensible candidate exists, return `blocked`
   - classify target-only fields deterministically:
     - optional target-only field with one defensible candidate -> `proposal-plus-approval`
     - optional target-only field with omission as a defensible strategy and no stronger requirement -> `proposal-plus-approval`
     - optional target-only field with multiple plausible candidates -> `proposal-plus-approval`
     - required target-only field with one defensible candidate -> `proposal-plus-approval`
     - required target-only field with multiple plausible candidates, or with no defensible candidate -> `blocked`
   - silent omission is never a valid outcome for a target-only field; omission must appear as an explicit analyzed proposal when allowed
   - do not treat examples/defaults/enums as autonomous authorization to invent new values
   - classify body changes as:
     - `safe-autonomous` when field carry-forward is direct and requires no semantic invention
     - `proposal-plus-approval` when one plausible rename, move, nesting change, type-conversion, value source, or target-only-field proposal exists but semantic confirmation is still required
     - `blocked` when multiple plausible target fields compete, structural reshaping is ambiguous, one target field appears to derive from multiple source fields, a required target field has no trustworthy value source, or body analysis could not be completed
   - illustrative review triggers:
     - a flat source field moves under a new nested target object (for example `phoneNumber` -> `contactInformation.phoneNumber`)
     - the target adds a new field with no source equivalent (for example a new `email` field)
7. Analyze supported downstream chained tools:
   - use Skills 017 and 018 as the canonical output-parent and chaining authority
   - keep family-selection and repair-confidence decisions here rather than rediscovering them inside downstream tool cards
   - partition downstream cases into:
     - selector-bearing repairs (JSON Data Bank / JSON Assertor)
     - validator retargeting repairs (JSON Validator)
     - comparison/baseline repairs (Diff Tool)
     - pre-existing suspect chains that should be reported rather than silently “fixed” as migration-caused drift
   - preserve canonical parent semantics:
     - semantic JSON validation/extraction remains anchored to `Response Traffic`
     - sibling `Traffic Viewer` presence is not evidence of the correct repair parent
   - apply repair-confidence rules:
     - `safe-autonomous` when selector/validator/comparison mode preservation is mechanically deterministic
     - `proposal-plus-approval` when one plausible selector rewrite, validator retargeting, or baseline-refresh strategy exists but semantic confirmation is still required
     - `blocked` when referenced response fields disappear with no clear equivalent, family replacement would be required, or comparison/assertion intent is no longer clear
   - produce one repair-intent record per supported downstream tool containing at least:
     - tool identity
     - tool family
     - repair kind
     - old selector/binding/baseline anchor when relevant
     - new selector/binding/baseline proposal when known
     - approval state required before write mode can apply the repair
8. Emit client-local unresolved decision items for:
   - operation ambiguity
   - parameter ambiguity
   - body ambiguity
   - chained-tool selector/validator/baseline ambiguity
   - each item should carry enough detail for orchestration-level grouping without recomputing the original client-local reasoning
   - for request-body items specifically, emit the structured body-review fields from the result contract rather than reducing them to free-form prose
9. Return the structured `analysis`-mode result contract for orchestration-level grouping.

### 6.2) Write Mode
10. Require resolved client-local decisions before any mutation:
   - if unresolved items remain, stop and return `blocked`
10.1. Enforce the body-analysis precondition:
   - if the source or target operation has a request body, require an explicit request-body mapping result and a non-`NOT_APPLICABLE` `requestBodyState`
   - do not treat source-spec/base URL rewrites or operation-match evidence as substitutes for completed request-body analysis
11. Build a concrete client-local write plan from:
   - the approved/autonomous operation mapping
   - the approved/autonomous parameter decisions
   - the approved/autonomous body decisions
   - the approved/autonomous downstream repair contracts
   - preserve dependency order so downstream tool edits do not race ahead of unresolved client request/response shape changes
12. Apply exact in-scope client refactor changes using Skill 059 mechanics where applicable:
   - keep path + method as the authoritative operation identity
   - preserve `baseUrl` unless upstream orchestration already approved an exact-match base URL rewrite policy for this branch
   - rewrite only exact in-scope source-spec references for this client slice
   - when the approved change includes constrained `Form JSON` request-body edits, route that branch through Skill 059's validated REST Client `GET/PUT/GET` body-normalization path rather than direct persisted YAML body-tree mutation
   - use validated constrained JSON request-body normalization paths rather than hand-inventing unsafe persisted shapes
   - fail closed if the required client mutation falls outside the validated constrained lifecycle mechanics available beneath this leaf
13. Apply approved/autonomous parameter and request-body changes in client-local dependency order:
   - keep query-parameter mirrored surfaces synchronized when query changes are applied
   - apply only:
     - autonomous defaults explicitly allowed by this card, or
     - proposal/blocked items that have already been resolved upstream
   - do not silently invent new required parameter/body values during write mode
   - do not silently drop unresolved target-only fields; apply only approved candidate-value or omission decisions
14. Apply supported downstream chained-tool repairs using precomputed repair contracts:
   - JSON Validator
   - JSON Data Bank
   - JSON Assertor
   - Diff Tool
   - downstream tool-family cards may be used for payload/config semantics, but do not let them reopen already-settled family selection or approval logic
   - preserve canonical parent semantics from Skills 017 and 018
   - do not silently substitute an unsupported tool family when the approved repair intent cannot be delivered as written
15. Perform client-slice structural verification:
   - confirm the client subtree remains structurally coherent in the copied `.tst`
   - confirm updated downstream tool nodes remain resolvable under the intended parent structure
   - confirm the leaf can still read back the expected client-local structures after mutation
15.1. Perform semantic body verification for body-bearing clients whose approved plan changed the request body:
   - confirm the copied constrained client now exposes the approved target field paths/shapes rather than only rewritten source-spec/base-url references
   - do not treat source-spec/base URL rewrites or generic file readability as evidence that request-body migration succeeded
16. If client-slice structural verification fails:
   - roll back that client slice
   - return `rolled back`
17. If structural verification succeeds:
   - keep the client-slice changes
   - return the structured `write`-mode result contract

## 7) Validation
- `analysis` mode produces a structured per-client result without mutating the source or copied `.tst`.
- `analysis` mode emits explicit per-domain decision states for every applicable domain.
- `requestBodyState=NOT_APPLICABLE` is used only when both source and target operations have no request body.
- For body-bearing operations, the required request-body result/state must be present; path+method continuity alone is never sufficient for overall `AUTO`.
- When target-only body fields lack direct source mapping, analysis surfaces explicit candidate-or-omit review material when defensible, otherwise `blocked`.
- `write` mode does not begin until client-local unresolved decisions have been resolved upstream.
- Path + HTTP method remains the authoritative operation identity for one-client refactor reasoning.
- Supported downstream tool repair remains limited to:
  - JSON Validator
  - JSON Data Bank
  - JSON Assertor
  - Diff Tool
- `write` mode applies only approved/autonomous decisions; it does not reopen unresolved mapping logic or invent new required values.
- Client-slice structural verification is required before the slice is considered applied.
- When approved request-body migration work exists, semantic body verification must confirm the copied client reflects the approved target field paths/shapes; structural readability alone is insufficient.
- Failed client-slice structural verification results in rollback rather than leaving behind a known-broken partial subtree.

## 8) Failure Modes
- Source client is not actually within the validated constrained REST Client boundary.
- Source-spec binding does not exactly match the current migration slice.
- Operation mapping remains ambiguous or unsupported.
- Exact path+method match is allowed to collapse a body-bearing client to `AUTO` before explicit request-body analysis is completed.
- `requestBodyState` is marked `NOT_APPLICABLE` even though the source or target operation has a request body.
- Required target parameter/body values have no defensible source.
- Target-only body fields are silently dropped instead of surfacing as explicit review items with candidate or omission proposals.
- Write completion is inferred from source-spec/base-url rewrites or file readability without confirming the approved target request-body shape.
- Downstream tool repair would require unsupported family conversion.
- Downstream lifecycle cards attempt to reopen family selection or user-facing approval that should already have been decided upstream.
- Write mode tries to apply decisions that were not resolved during analysis/grouped review.
- Seeing persisted constrained `Form JSON` body structures in downloaded YAML is treated as authorization to mutate that body tree directly instead of routing the body branch through Skill 059 REST Client normalization.
- Client-slice write changes leave the copied `.tst` structurally incoherent.

## 9) Safety / Rollback
- This is a write-capable leaf in `write` mode and must remain fail-closed.
- Treat the client subtree as the transaction boundary when possible.
- If the client subtree cannot be left structurally coherent, restore that client slice and return `rolled back`.
- Do not convert client-slice failure into a silent partial success.

## 10) Reuse Notes
- This card is intended as the lower-layer one-client refactor owner beneath broader orchestration such as `Change Advisor`.
- It is adjacent to Skill 059, but it does not replace Skill 059:
  - Skill 059 owns validated constrained REST Client lifecycle mechanics
  - Skill 060 owns one-client source-to-target refactor reasoning plus supported downstream repair ownership
- Use the validation/data-exchange cards primarily as tool-family semantic references in this leaf; do not require a broader lifecycle-vs-action-contract refactor before using this card.
