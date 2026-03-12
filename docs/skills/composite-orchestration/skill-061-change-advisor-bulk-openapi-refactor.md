# Skill 061: Change Advisor (Composite)

## 1) Skill Name
Orchestrate bulk constrained REST Client OpenAPI refactor work for one confirmed source-spec slice in one existing `.tst`.

## 2) Objective
Convert direct bulk version-to-version OpenAPI refactor intent into a confirmed, staged workflow that:
- normalizes intake for one source `.tst`, one confirmed source OpenAPI anchor, and one confirmed target OpenAPI anchor
- performs autonomous read-only migration inventory and one-client refactor analysis
- presents a structural approval artifact plus grouped review packet
- copies the source `.tst` and applies approved writes only against the copied target asset
- returns a concise final completion artifact with structural outcome and follow-up hotspots

This card is the operator-facing owner of the bulk refactor workflow. It delegates one-client source-to-target refactor semantics to Skill 060 and does not absorb those lower-layer mechanics into the orchestration layer.

Conversation-first default: favor short natural-language clarification over rigid choice widgets when intake inputs or grouped review decisions are still unresolved.

## 3) Scope
- In scope:
  - direct bulk OpenAPI refactor intent on one existing `.tst`
  - one confirmed source OpenAPI -> one confirmed target OpenAPI migration slice at a time
  - constrained REST Clients as the v1 migrated executable family
  - mandatory read-only analysis and grouped review followed by mutate-on-copy-after-approval writes on a copied target `.tst`
  - intake normalization and v1 boundary confirmation
  - read-only whole-file inventory and exact source-spec slice selection
  - per-client delegation to Skill 060 in `analysis` mode
  - grouped post-analysis review ownership across all in-scope clients
  - copied-target naming and copied-target write sequencing
  - exact source-spec reference rewrites for the approved migration slice
  - per-client delegation to Skill 060 in `write` mode
  - final copied-artifact structural verification and completion reporting
- Out of scope:
  - one-client refactor intent that already belongs directly to Skill 060
  - unconstrained REST migration
  - SOAP, DB Tool, or mixed non-REST migration as first-class executable families
  - datasource refactoring
  - XML-family chained-tool refactor ownership in v1
  - fully autonomous semantic conflict resolution
  - runtime-debugging ownership after the bulk refactor workflow has completed

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
  - `docs/skills/platform/skill-003-server-copy.md`
  - `docs/skills/client-tools/skill-060-single-constrained-rest-client-openapi-refactor.md`
- Additive:
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`

## 4) Inputs
- Required:
  - user objective to bulk-refactor an existing `.tst` from one OpenAPI contract/version/location to another
  - source `.tst` target
  - exact current/source OpenAPI anchor
  - exact target OpenAPI anchor
- Optional:
  - explicit subset override within the source-spec slice
  - target copied-asset naming preference
  - target copied-asset location preference
  - explicit base URL policy preference:
    - keep the current/source base URL
    - rewrite to the target OpenAPI server/base URL when available
    - rewrite to a user-supplied base URL
  - explicit deferment of chained-tool repair
  - desired success emphasis:
    - structural migration only
    - runtime-ready follow-up planning after structural migration

## 4.1) Working Records
This card should maintain structured records between phases rather than relying only on prose summaries.

### Migration Target Record
Each in-scope constrained REST Client should produce one migration target record containing:
- source `.tst` path context
- source client identity
- source-spec binding evidence
- target operation mapping status and target operation identity when known
- parameter mapping summary
- request-body mapping summary
- chained-tool impact summary
- overall decision-state summary used by the structural marker (`AUTO`, `REVIEW`, or `BLOCKED`)

### Review Item Record
Each unresolved review item should contain:
- stable review id
- owning migration target reference, or `workflow-global` when the item applies to the whole run
- mapping layer (`operation`, `parameter`, `body`, `chained-tool`)
- status (`REVIEW` or `BLOCKED`)
- dependency references when applicable
- decision prompt text
- agent proposal text when one exists
- rationale text explaining why review is required
- current lifecycle state (`pending`, `resolved`, `superseded`, `still-blocked`)
### Run Policy Record
Maintain a workflow-global run-policy record containing:
- current/source base URL evidence for the in-scope slice
- candidate target OpenAPI server/base URL values when available
- current base URL policy status:
  - `pending user choice`
  - `keep current/source`
  - `rewrite to target OpenAPI server/base URL`
  - `rewrite to user-supplied base URL`
- the final approved base URL value when a rewrite policy is chosen

### Write Result Record
After writes, each migration target should yield one write result record containing:
- copied target `.tst` id/path
- migration target reference
- whether exact source-spec reference rewrites were applied
- whether constrained REST refactor was applied
- whether chained-tool repair was applied
- structural verification status
- final state classification (`applied`, `partial`, `blocked`, `rolled back`)
- follow-up recommendation text when further focused execution or manual iteration is advised

## 4.2) Evidence Boundary
- Before the source `.tst` is downloaded, API reads are allowed only for:
  - exact source `.tst` resolution
  - exact source/target OpenAPI retrieval
  - capability confirmation for later copy/write routes
- After the source `.tst` download begins, refactor-analysis evidence must come from:
  - the downloaded source `.tst`
  - normalized source/target OpenAPI structures
  - structured Skill 060 `analysis` results for in-scope clients
- Do not use ad hoc post-download API rediscovery of constrained REST internals, copied-target content, or transient local experiment files as competing analysis sources of truth.
- If chained-tool repair is explicitly deferred, keep the deferred impact visible in the approval/completion artifacts rather than silently dropping it from the analysis story.

## 5) Preconditions
- API reachable and authenticated for all required downstream skills.
- The source `.tst` is resolvable exactly on the server.
- The source and target OpenAPI anchors are resolved enough to retrieve and normalize them.
- The source OpenAPI anchor exactly matches the in-scope constrained REST Client bindings before bulk analysis continues.
- `write` phase must not begin until:
  - grouped review items are resolved or explicitly deferred within the approved scope, and
  - the user explicitly confirms proceeding with writes on a copied target `.tst`.

## 6) Procedure
### 6.1) Intake Normalization
1. Confirm the request is bulk refactor intent on an existing `.tst`, not:
   - broad service-test generation -> route to Skill 033
   - one-client refactor work -> route to Skill 060
2. If the user requests unsupported v1 scope, stop and classify it explicitly rather than silently broadening the workflow. Unsupported-v1 examples include:
   - unconstrained REST migration
   - SOAP or DB migration as first-class executable scope
   - multi-spec or multi-target bulk migration in one run
   - datasource refactoring
   - fully autonomous semantic conflict resolution
3. Resolve the source `.tst`.
4. Resolve the exact current/source OpenAPI anchor and exact target OpenAPI anchor.
5. If the source `.tst`, source OpenAPI anchor, and target OpenAPI anchor are not all resolved, remain in clarification mode and do not begin migration analysis.
6. Present the v1 workflow boundaries clearly before deeper analysis:
   - constrained REST only
   - one source-spec slice at a time
   - non-target families preserved unless they appear in scoped chained-tool repair context
   - base URL will not change silently; any rewrite requires explicit approval
   - copied-target-only writes
7. Resolve optional intake values only when needed:
   - default full eligible source-spec slice vs explicit subset override
   - target copied-asset naming and location preferences
   - whether the user already has an explicit base URL policy preference; otherwise defer that choice to the post-analysis approval artifact once current/source and target-candidate values are known
   - whether chained-tool repair should be explicitly deferred
   - desired success emphasis
   - if an explicit subset override is requested, resolve it to a stable client list by exact test path/client identity before analysis; if the override is ambiguous, stop and clarify rather than guessing the subset
8. Build and present a concise intake summary containing:
   - source `.tst`
   - confirmed v1 assumptions/boundaries
   - confirmed source OpenAPI anchor
   - confirmed target OpenAPI anchor
   - source OpenAPI verification status:
     - `pending exact-match check in read-only analysis gate`, or
     - `already verified`
   - base URL policy status:
     - `pending approval-step choice unless already specified`, or
     - the user-specified policy
   - default full eligible source-spec slice vs explicit subset override
   - target copied-asset naming and location preferences
   - chained-tool repair in scope vs explicitly deferred
   - desired success emphasis
   - warning that explicitly deferred chained-tool repair may leave the copied output structurally usable but semantically incomplete
   - next phase to run
9. Require confirmation of the intake summary before starting the read-only migration analysis phase.

### 6.1.1) API Capability and Resolution Gate (Required)
8. Before any write operation, run branch-aware capability preflight (Skill 050):
  - apply the operation-class profile from Skill 050 Section 6.1
  - classify outcomes and `404` disambiguation per Skill 050 Section 6.2
  - record/reuse capability outcomes per Skill 050 Section 6.3
9. During the read-only planning phase, perform only the capability checks required to confirm:
  - source `.tst` resolution
  - download compatibility
  - copy/write route availability for the later mutate-on-copy phase
10. Do not continue past the intake gate when the source `.tst` or either OpenAPI anchor cannot be resolved or retrieved reliably.

### 6.2) Read-Only Migration Analysis
11. Resolve the exact rooted source `.tst` id and download the file locally through Skill 002.
12. Treat the downloaded `.tst` as the analysis source of truth for constrained REST refactor evidence.
13. Do not mix ad hoc post-download API rediscovery of constrained REST internals into the refactor-analysis evidence model.
14. Parse a local structural inventory that includes:
  - suites
  - scenarios
  - tests
  - constrained REST Clients
  - chained tools
  - environments
  - datasource references
15. Discover all source OpenAPI locations referenced by constrained REST Clients in scope, including:
  - hardcoded literals
  - `${TOKEN}` references
  - other YAML-visible indirection resolvable from the downloaded `.tst`
16. Require an exact match between the user-provided source OpenAPI anchor and the discovered source-spec binding evidence.
17. If exact-match verification fails, stop and ask for correction rather than inferring the intended migration slice.
18. Select as in-scope only those constrained REST Clients whose source-spec binding exactly matches the confirmed source OpenAPI anchor and any approved subset override.
   - preserve stable source-tree order for the in-scope client list and reuse that same ordering for analysis presentation, review-item ordering, and write sequencing unless a later dependency rule explicitly overrides it
19. For each in-scope constrained REST Client, record:
  - test path context
  - current operation identity by OpenAPI path + HTTP method
  - current request/body mode
  - schema/base-url reference style
  - immediate chained-tool descendants
20. Retrieve and normalize the exact source and target OpenAPI documents into operation, parameter, request-schema, and response-schema maps.
20.1. Build the workflow-global base URL evidence record before the approval artifact:
  - capture the current/source base URL values used by the in-scope slice (for example environment variables and client-local base URL references)
  - derive candidate target OpenAPI server/base URL values from the target OpenAPI when the contract exposes them
  - if the user already supplied a base URL policy, record it; otherwise keep the policy state as `pending user choice`
21. For each in-scope client, invoke Skill 060 in `analysis` mode and capture its structured per-client result.
22. Convert each Skill 060 analysis result into a migration target record.
23. Convert each client-local unresolved decision into a centralized review item record with whole-run review ids.
   - keep review ids stable in source-tree order for the first packet
   - if chained-tool repair was explicitly deferred, do not generate write-intent review items for those repairs; instead record them as explicit deferred hotspots tied to the owning migration target
24. If analysis reveals gate-invalid conditions such as ambiguous source binding, unretrievable contracts, or unsupported client state before trustworthy comparison is possible, stop before write preparation and report the planning result as blocked.
   - distinguish global gate failure from client-local blocked targets:
     - global gate failure stops the whole workflow before the approval artifact
     - client-local blocked targets still appear in the approval artifact as blocked migration targets when the overall slice analysis remains valid

### 6.3) Structural Approval Artifact and Grouped Review Packet
25. After all eligible clients have been analyzed, build one operator-facing approval artifact containing:
  - source `.tst`
  - source OpenAPI
  - target OpenAPI
  - current/source base URL evidence and any target OpenAPI base URL candidate
  - current base URL policy status
  - a short note about any blocked or ambiguous planning hotspots
  - a structural tree view
  - a grouped review packet
26. The structural tree must use these marker rules:
  - `[AUTO]` for refactor targets that can proceed without more user input
  - `[REVIEW]` for refactor targets needing user confirmation
  - `[BLOCKED]` for refactor targets needing user direction before refactor can proceed
  - unmarked nodes are context only and are not refactor targets
27. Apply worst-child-wins inheritance for marked parent nodes:
  - any in-scope blocked descendant -> parent becomes `[BLOCKED]`
  - else any in-scope review descendant -> parent becomes `[REVIEW]`
  - else all in-scope descendants autonomous -> parent becomes `[AUTO]`
28. Keep the tree selective rather than dumping the entire `.tst`:
  - expand only relevant source-spec branches
  - omit top-level `Data Sources` and `Environments` by default unless they are directly needed to explain the migration slice or a scoped repair/deferment decision
  - collapse irrelevant sibling branches to one context line
  - show Traffic Viewer only as unmarked context
  - allow other non-target executable/context branches to remain unmarked when they help explain the surrounding scenario structure
  - when the active scenario contains adjacent non-target executable context such as a `DB Tool`, prefer rendering that node as unmarked context rather than omitting it, because it helps orient the user to the local subtree even though it is outside the v1 refactor family
29. Use `docs/templates/change-advisor-approval-artifact-template.md` as the canonical output contract for the initial post-analysis approval artifact.
30. In that template, the initial grouped review packet belongs inside the same approval artifact as the structural tree, not as a separate first response.
31. Present grouped review items in this order:
  - workflow-global run-policy items first (for example base URL policy)
  - then by client and dependency order:
    - operation
    - path/query parameters
    - request body
    - chained tools
32. For each review item, emit the same compact fields in the same order:
  - review id + status
  - mapping layer
  - `Decision needed:`
  - `Agent proposal:` when one exists, otherwise `none`
  - `Why review is needed:`
  - `Depends on:` when applicable
33. If no review items remain after analysis, state that explicitly inside the template-owned review section and move to the final write-confirmation gate.
34. Do not duplicate or override template-owned section ordering, legend wording, or render-shape rules in this card.
   - if chained-tool repair was explicitly deferred, surface that as an explicit deferred hotspot in the template-owned planning note or review-area prose rather than hiding it
   - do not invent additional tree markers or use the tree body to carry detailed blocked reasoning that belongs in the grouped review section

### 6.4) Review-Packet Resolution Loop
35. Treat the grouped review packet as a resolve-recompute-re-render loop, not as a one-shot prose exchange.
36. For `[REVIEW]` items, accept as sufficient:
  - approval
  - rejection with replacement direction
  - direct replacement mapping/value
37. For `[BLOCKED]` items, bare approval or rejection is not sufficient. Require concrete direction such as:
  - a replacement mapping
  - a value source
  - a strategy choice
38. If a user response changes an upstream dependency materially:
  - mark dependent items `superseded`
  - recompute only the affected client-local reasoning
  - issue new review ids for new unresolved items
39. After the initial approval artifact, reduced follow-up packets may be emitted as separate review-only responses containing only unresolved or newly created review items unless the structural decision-state view changed materially enough to justify reprinting the full artifact.
   - when helpful, include brief resolution notes and supersession notes so the user can see what changed without re-reading the entire first artifact
40. Do not start writes only because some review items were answered. The write phase may begin only after:
  - all in-scope review items are resolved or explicitly deferred within the approved scope
  - no recomputation has produced new unresolved items
  - the user explicitly confirms proceeding with writes

### 6.5) Write-Phase Orchestration
41. After the approval artifact and grouped review resolution path, begin writes only when the user explicitly confirms proceeding with mutate-on-copy-after-approval.
42. Create a copied target `.tst` before any refactor writes.
43. By default, create the copied target in the same rooted server directory/location as the source `.tst`. If the user supplied an explicit copy-location preference, use it only after the destination path is resolved exactly and is valid for server-side copy behavior.
44. Use this default copied-target naming strategy unless the user approved another name:
  - append `-refactored` before the extension
  - if occupied, suffix numerically without an extra dash: `-refactored2`, `-refactored3`, and so on
45. Preserve the original source `.tst` untouched throughout the workflow.
46. Before per-client writes, rewrite only exact in-scope source-spec references for the confirmed migration slice in the copied target `.tst`, including:
  - environment-variable values whose exact value matches the confirmed source OpenAPI anchor
  - constrained REST Client-local schema/service-definition references whose exact value matches the confirmed source OpenAPI anchor
  - chained validator or related configuration nodes whose hardcoded definition source exactly matches the confirmed source OpenAPI anchor
   - if the same exact source OpenAPI anchor appears in multiple environments, rewrite each exact match, not just the active environment
   - leave non-matching references for other spec slices unchanged
47. Apply the explicitly approved base URL policy after exact source-spec rewrites:
  - `keep current/source`: preserve existing base URL values unchanged
  - `rewrite to target OpenAPI server/base URL`: rewrite only exact in-scope source base URL matches to the approved target OpenAPI server/base URL value
  - `rewrite to user-supplied base URL`: rewrite only exact in-scope source base URL matches to the user-supplied value
  - do not silently change base URLs, and do not infer a rewrite target when the target OpenAPI does not expose a usable server/base URL
48. Apply per-client writes in stable source-tree order unless an explicit dependency or safety reason requires a different order.
48.1. Before invoking Skill 060 in `write` mode for one copied target client, re-resolve the copied client location deterministically using Skill 001-style asset-graph targeting rather than assuming the source nested tool id still applies:
  - resolve the copied `.tst` root suite from the copied file id
  - walk the stored suite/scenario/test path context captured during read-only analysis
  - within the resolved copied local subtree, select the intended `restClient` by expected type plus stable local identity (for example preserved name/label), using preserved source-tree order only as a tie-breaker when needed
  - if the copied local subtree cannot be re-established exactly or multiple candidate tools remain, stop and report that client slice as blocked rather than guessing a copied tool id
49. For each approved/unblocked migration target, invoke Skill 060 in `write` mode against the copied target `.tst`.
50. Capture each Skill 060 write result as a write result record.
   - if chained-tool repair was explicitly deferred, still apply the approved constrained REST refactor work for those clients, but record the downstream repair state as deferred rather than silently treating the client as fully complete
51. If one client slice rolls back but the copied target `.tst` remains structurally valid, keep the copied target and continue only when doing so will not leave the whole file structurally incoherent.
52. Do not silently convert a blocked or rolled-back client slice into a success; report it as `partial`, `blocked`, or `rolled back` in the final artifact.

### 6.6) Final Structural Verification and Completion Artifact
53. After all intended writes complete, verify the copied target `.tst` at the file level:
  - readback succeeds
  - the copied artifact remains parseable/openable as a `.tst`
54. If the copied target `.tst` is corrupt, unreadable, or structurally invalid:
  - fail closed
  - restore or replace the copied target with the last structurally valid copied state
  - report the bulk write result as rolled back
55. If the copied target `.tst` remains structurally valid but some refactor steps were blocked, partial, or rolled back at the client-slice level:
  - keep the copied target `.tst`
  - report the unresolved hotspots clearly
56. Use `docs/templates/change-advisor-completion-artifact-template.md` as the canonical output contract for the final completion artifact.
57. The final completion artifact must include:
  - original source `.tst`
  - copied refactored `.tst`
  - source OpenAPI -> target OpenAPI
  - count of targeted clients processed
  - count of review items resolved
  - success tier status:
     - structural success
     - mechanical migration success
     - runtime-readiness status
     - semantic-confidence status
  - final structural verification result
  - whether the copied `.tst` was kept or rolled back
  - partial/blocked follow-up hotspots
  - recommended next step: focused execution and traffic confirmation of changed tests
58. Carry the desired success emphasis into the final artifact and closing summary:
   - `structural migration only` emphasizes structural success plus mechanical migration outcome while still recommending focused execution as a later follow-up
   - `runtime-ready follow-up planning` emphasizes the changed-test hotspot list and immediate focused execution/traffic confirmation as the next collaboration step
59. Do not duplicate or override template-owned section ordering or field-shape rules in this card.
60. Keep focused execution and runtime-debugging work as an explicit follow-up collaboration rather than silently extending this card into long-running post-refactor diagnostics.

## 7) Validation
- Bulk refactor intent is distinguished from one-client refactor intent before execution.
- Unsupported-v1 scope is explicitly classified and stopped rather than silently broadened.
- Migration analysis does not begin until the source `.tst`, source OpenAPI anchor, and target OpenAPI anchor are all resolved.
- Read-only analysis does not mutate the source or copied `.tst`.
- Source-spec slice selection is based on exact binding evidence, not loose similarity.
- Whole-run review ids and grouped review ownership remain centralized in this card.
- Skill 060 owns one-client refactor semantics; this card does not reopen that lower-layer reasoning.
- The initial approval artifact is built from the downloaded source `.tst`, normalized source/target OpenAPI structures, and Skill 060 analysis results rather than mixed competing evidence sources.
- Writes occur only against a copied target `.tst`.
- Base URL behavior is explicit and approval-bound: preserved only when the approved policy says to keep it, otherwise rewritten only according to the approved exact-match policy.
- Explicit subset overrides and explicitly deferred chained-tool repair remain visible and deterministic in the approval/completion artifacts.
- Structural verification is the mandatory completion gate for the copied artifact.
- The final artifact reports the plan's success tiers rather than reducing the outcome to one binary success/failure statement.
- Parseable-but-partial copied results are kept and reported; corrupt copied artifacts are rolled back.

## 8) Failure Modes
- One-client refactor intent is incorrectly forced through the bulk card.
- Source and target OpenAPI anchors are assumed instead of confirmed.
- Unsupported-v1 requests are allowed to drift into this workflow instead of being classified and stopped.
- Mixed or contradictory source-spec bindings are treated as one migration slice.
- Planning proceeds from API-only constrained-client evidence instead of the downloaded `.tst` source of truth.
- Review items are asked one-by-one during analysis instead of being grouped after analysis.
- Blocked review items are treated as resolved from bare approval.
- Explicit subset overrides are guessed rather than resolved to a stable target list.
- Explicitly deferred chained-tool repair disappears from the artifact and final report instead of being carried as a visible deferred hotspot.
- Relevant non-target executable context (for example a same-scenario `DB Tool`) is omitted from the approval tree even though it would help orient the user to the local subtree.
- The copied-target destination is assumed implicitly when the user requested a different copy location.
- Copied nested tool ids are assumed to survive file copy, or are guessed from one broad descendants query, instead of being re-resolved from the copied root using stable path context.
- The final report collapses structural, mechanical, runtime-readiness, and semantic-confidence outcomes into one ambiguous success label.
- The top-level card absorbs one-client mutation semantics that should remain in Skill 060.
- Writes begin before copied-target creation or before final explicit write confirmation.
- The workflow fails to request an explicit base URL policy choice and either preserves or rewrites base URLs without approval.
- Base URLs are rewritten implicitly or from a guessed target value rather than from the explicitly approved policy.
- A client-slice rollback is misreported as full workflow success.

## 9) Safety / Rollback
- This is a write-capable composite and must remain fail-closed.
- The original source `.tst` is the primary safety baseline and must remain untouched.
- The copied target `.tst` is the only write target.
- Structural corruption of the copied target requires restore/replace with the last structurally valid copied state.
- Parseable partial outcomes may be kept, but they must be reported explicitly as partial rather than silently promoted to success.

## 10) Reuse Notes
- Use this card for direct bulk version-to-version OpenAPI refactor work on an existing `.tst`.
- Use Skill 060 instead when the user already wants exactly one constrained REST Client subtree analyzed or refactored.
- Use Skill 033 when the request is still broad service-test authoring/generation rather than refactor intent on an existing `.tst`.
- This card is the operator-facing owner of grouped review, copied-target sequencing, and final completion reporting for the bulk refactor path.
