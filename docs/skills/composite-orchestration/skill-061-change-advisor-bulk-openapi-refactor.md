# Skill 061: Change Advisor (Composite)
## 1) Skill Name
Orchestrate bulk constrained REST Client OpenAPI refactor work for one confirmed source-spec slice in one existing `.tst`.
## 2) Objective
Convert direct bulk version-to-version OpenAPI refactor intent into a confirmed, staged workflow that:
- normalizes intake for one source `.tst`, one confirmed source OpenAPI anchor, and one confirmed target OpenAPI anchor
- performs autonomous read-only migration inventory and one-client refactor analysis
- applies posture-aware review handling:
  - interactive mode uses a structural approval artifact plus grouped review packet
  - autonomous mode auto-accepts agent proposals for `[REVIEW]` items and interrupts only for true blockers
- creates a local copied target `.tst` and applies resolved writes only against that copy
- returns a concise final completion artifact with structural outcome and follow-up hotspots
This card is the operator-facing owner of the bulk refactor workflow. It delegates one-client refactor semantics to Skill 060.
## 3) Scope
- In scope:
  - direct bulk OpenAPI refactor intent on one existing local `.tst`
  - one confirmed source OpenAPI -> one confirmed target OpenAPI migration slice at a time
  - constrained REST Clients as the v1 migrated executable family
  - mandatory read-only analysis plus posture-aware review handling followed by mutate-on-copy writes on a copied target `.tst`
  - intake normalization, exact source-spec slice selection, grouped review/blocker handling, copied-target naming, local copy/write sequencing, and final completion reporting
- Out of scope:
  - one-client refactor intent that belongs directly to Skill 060
  - unconstrained REST migration
  - SOAP, DB Tool, or mixed non-REST migration as first-class executable families
  - fully autonomous resolution of `[BLOCKED]` semantic conflicts without user direction
  - runtime-debugging ownership after the bulk refactor workflow has completed
## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/client-tools/skill-060-single-constrained-rest-client-openapi-refactor.md`
- Additive:
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md` when active project context is available
## 4) Inputs
- Required:
  - user objective to bulk-refactor an existing `.tst` from one OpenAPI contract/version/location to another
  - source `.tst` local path or enough file-backed identity to resolve it through Skill 001
  - exact current/source OpenAPI anchor
  - exact target OpenAPI anchor
- Optional:
  - target copied-asset naming preference
  - target copied-asset location preference
  - explicit base URL policy preference
  - explicit deferment of chained-tool repair
  - autonomous/no-interrupt execution posture
  - desired success emphasis
## 4.1) Working Records
Maintain structured records between phases:
- Migration Target Record
- Review Item Record
- Run Policy Record
- Write Result Record
## 4.2) Evidence Boundary
- Before the source `.tst` local file read begins, only local target resolution plus source/target OpenAPI retrieval are allowed.
- After the source `.tst` local file read begins, refactor-analysis evidence must come from:
  - the local source `.tst`
  - normalized source/target OpenAPI structures
  - structured Skill 060 `analysis` results for in-scope clients
- Runtime/API reads are allowed later only for:
  - explicit Skill 050 preflight on approved API branches
  - runtime-object resolution needed by delegated write branches
  - API readback verification of approved API-normalized body branches
- Do not use ad hoc runtime rediscovery of constrained REST internals as competing analysis sources of truth.
## 5) Preconditions
- The source `.tst` local path is resolved.
- The source and target OpenAPI anchors are resolved enough to retrieve and normalize them.
- The source OpenAPI anchor exactly matches the in-scope constrained REST Client bindings before bulk analysis continues.
- `write` phase must not begin until posture-specific review gates are satisfied:
  - interactive mode: grouped review items are resolved or explicitly deferred within approved scope and the user explicitly confirms proceeding with writes
  - autonomous mode: all `[BLOCKED]` items are resolved and `[REVIEW]` items are auto-accepted using agent proposals unless the user already overrode them
## 6) Procedure
### 6.1) Intake Normalization
1. Confirm the request is bulk refactor intent on an existing `.tst`, not broad generation or one-client refactor work.
2. If the user requests unsupported v1 scope, stop and classify it explicitly rather than silently broadening the workflow.
3. If execution posture is unresolved, ask exactly:
   - `Do you want me to create your tests autonomously and only interrupt if I'm blocked, or do you want to stay interactive while we shape the experimental plan together? Just say 'autonomous' or 'auto', or 'interactive'.`
4. Resolve the source `.tst` locally through Skill 001.
5. Resolve the exact current/source OpenAPI anchor and exact target OpenAPI anchor. When active project context is available, consult project references/facts first before asking the user to restate known contract anchors or sanctioned target locations.
6. Normalize the run against these v1 boundaries:
   - constrained REST only
   - one source-spec slice at a time
   - analyze all constrained REST Clients whose binding matches the confirmed source OpenAPI anchor
   - no subset-selection intake for that exact-match slice
   - base URL will not change silently; any rewrite still requires either explicit interactive approval or autonomous auto-acceptance of the agent proposal
   - copied-target-only writes
7. Once required inputs are resolved, proceed directly to read-only migration analysis.
### 6.1.1) Execution Posture Contract
- Interactive mode:
  - present the structural approval artifact plus grouped review packet
  - resolve `[REVIEW]` items through user decisions
  - require explicit write-phase confirmation after review resolution
- Autonomous mode:
  - treat `[REVIEW]` items as auto-accepted using the agent proposals unless the user already provided overrides
  - do not interrupt with the full approval artifact when issues are only `[AUTO]` and `[REVIEW]`
  - interrupt only when at least one `[BLOCKED]` or gate-invalid blocked condition exists
  - when blocked, present only the blocker packet and required blocker questions, then continue after resolution without reopening normal review summaries
### 6.1.2) API Capability Gate (Required)
8. Do not enter Skill 050 during read-only analysis unless an API branch actually becomes necessary.
9. Before any runtime-id discovery or mutation branch in the write phase, enter Skill 050 with the matching read or write/execution lane.
### 6.2) Read-Only Migration Analysis
10. Read the source `.tst` locally and treat it as the analysis source of truth.
11. Parse a local structural inventory that includes suites, scenarios, tests, constrained REST Clients, chained tools, environments, and datasource references.
12. Discover all source OpenAPI locations referenced by constrained REST Clients in scope from local YAML evidence.
13. Require an exact match between the user-provided source OpenAPI anchor and the discovered source-spec binding evidence.
14. Select as in-scope all constrained REST Clients whose source-spec binding exactly matches the confirmed source OpenAPI anchor.
15. Retrieve and normalize the exact source and target OpenAPI documents into operation, parameter, request-schema, and response-schema maps.
16. Build the workflow-global base URL evidence record.
17. For each in-scope client, invoke Skill 060 in `analysis` mode and capture its structured per-client result.
18. Convert Skill 060 analysis results into migration target records and grouped review items.
19. If analysis reveals gate-invalid conditions such as ambiguous source binding or unsupported client state before trustworthy comparison is possible, stop before write preparation and report the planning result as blocked.
### 6.3) Structural Approval Artifact and Grouped Review Packet
20. After all eligible clients have been analyzed, branch review output by execution posture:
   - interactive mode: build one operator-facing approval artifact containing source/target anchors, base-url policy status, structural tree, and grouped review packet
   - autonomous mode with one or more `[BLOCKED]` items: present only a blocker packet with required decisions; do not present the normal full approval artifact
   - autonomous mode with only `[AUTO]` and `[REVIEW]` items: keep review records internal and continue without interruption
21. Use these marker rules in the structural tree and review records:
   - `[AUTO]`
   - `[REVIEW]`
   - `[BLOCKED]`
   - unmarked nodes are context only
22. Use `docs/templates/change-advisor-approval-artifact-template.md` as the canonical output contract for interactive approval artifacts.
23. Present grouped review items in this order when interactive review or blocker packets are rendered:
   - workflow-global run-policy items first
   - then by client and dependency order: operation, parameters, request body, chained tools
24. If no review items remain after analysis, state that explicitly and move to the posture-specific write-confirmation gate.
### 6.4) Review-Packet Resolution Loop
25. Treat the grouped review packet as a resolve-recompute-re-render loop.
26. In interactive mode, for `[REVIEW]` items accept approval, rejection with replacement direction, or direct replacement mapping/value.
27. In autonomous mode, treat `[REVIEW]` items as auto-accepted using the agent proposal unless the user already supplied an override.
28. For `[BLOCKED]` items in either posture, require concrete user direction before writes continue.
29. Write-phase gate by posture:
   - interactive mode: all in-scope review items must be resolved or explicitly deferred and the user must explicitly confirm proceeding
   - autonomous mode: proceed automatically once no unresolved `[BLOCKED]` items remain
### 6.5) Write-Phase Orchestration
30. Create a copied target `.tst` locally before any refactor writes.
31. By default, create the copy next to the source `.tst`; if the user supplied another location, use that exact resolved local destination. If active project context already defines a different sanctioned target area for refactor outputs and the user did not override it, keep the copy within that active project area.
32. Default copied-target naming:
   - append `-refactored` before the extension
   - if occupied, suffix numerically without an extra dash
33. Preserve the original source `.tst` untouched.
34. Before per-client writes, rewrite only exact in-scope source-spec references in the copied target `.tst`.
35. Apply the resolved base URL policy after exact source-spec rewrites:
   - interactive mode: user-approved policy
   - autonomous mode: auto-accepted agent proposal unless the user supplied an explicit override
36. Apply per-client writes in stable source-tree order unless an explicit dependency or safety reason requires a different order.
37. Before invoking Skill 060 in `write` mode for one copied target client, re-resolve the copied client location deterministically from the copied local `.tst` tree.
38. For each approved/unblocked migration target, invoke Skill 060 in `write` mode against the copied target `.tst`.
39. When a delegated branch needs runtime ids or API-normalized body mutation, enter Skill 050 only for that branch.
40. Capture each Skill 060 write result as a write result record.
41. If one client slice rolls back but the copied target `.tst` remains structurally valid, keep the copied target only when doing so will not leave the whole file structurally incoherent.
### 6.6) Final Structural Verification and Completion Artifact
42. After all intended writes complete, verify the copied target `.tst` at the file level:
   - local readback succeeds
   - the copied artifact remains parseable/openable as a `.tst`
43. For copied constrained REST Client slices and supported chained tools, the required post-write completion gate is persisted-structure verification:
   - local YAML readback confirms the approved exact source-spec rewrites persisted
   - local YAML readback confirms approved base URL rewrites persisted when a rewrite policy was approved
   - local YAML readback confirms approved request-body field paths/shapes for body-bearing clients
   - when an API-normalized body branch was used, API readback also confirms the approved persisted shape
44. If the copied target `.tst` is corrupt or structurally invalid, fail closed, restore or replace the copied target with the last structurally valid copied state, and report the bulk write result as rolled back.
45. Use `docs/templates/change-advisor-completion-artifact-template.md` as the canonical output contract.
46. Keep focused execution and runtime-debugging work as an explicit follow-up collaboration rather than silently extending this card into long-running post-refactor diagnostics.
## 7) Validation
- Bulk refactor intent is distinguished from one-client refactor intent before execution.
- Unsupported-v1 scope is explicitly classified and stopped rather than silently broadened.
- Migration analysis does not begin until the source `.tst`, source OpenAPI anchor, and target OpenAPI anchor are all resolved.
- Read-only analysis does not mutate the source or copied `.tst`.
- Source-spec slice selection is based on exact local/YAML binding evidence, not loose similarity.
- Whole-run review ids and grouped review ownership remain centralized in this card.
- Execution posture is resolved before analysis when not already explicit, using the canonical autonomous/interactive prompt and accepted responses (`autonomous`/`auto` or `interactive`).
- Interactive mode still requires explicit user confirmation before write phase.
- Autonomous mode auto-accepts `[REVIEW]` proposals and interrupts only for `[BLOCKED]` items.
- Skill 060 owns one-client refactor semantics.
- Writes occur only against a copied target `.tst`.
- Structural verification is the mandatory completion gate for the copied artifact.
- When delegated API-normalized body branches are used, completion remains structural/persisted-state only unless the user explicitly expands the workflow scope.
## 8) Failure Modes
- The card treats file-backed `.tst` identity as API-first instead of resolving the local path first.
- Planning proceeds from runtime/API-only constrained-client evidence instead of the local `.tst` source of truth.
- Review items are asked one-by-one during analysis instead of being grouped after analysis.
- Autonomous mode interrupts on `[REVIEW]`-only packets instead of auto-accepting the agent proposals and continuing.
- Autonomous mode silently resolves `[BLOCKED]` items without explicit user direction.
- The workflow asks the user to choose a subset of matching constrained REST Clients even though the confirmed bulk-refactor slice should include all exact-match clients by default.
- The copied local target location is assumed implicitly when the user requested a different copy location.
- Delegated leaf verification wording causes the workflow to drift into focused execution or runtime debugging even though this card is supposed to stop at structural completion.
## 9) Safety / Rollback
- This is a write-capable composite and must remain fail-closed.
- The original source `.tst` is the primary safety baseline and must remain untouched.
- The copied target `.tst` is the only write target.
- Structural corruption of the copied target requires restore/replace with the last structurally valid copied state.
## 10) Reuse Notes
- Use this card for direct bulk version-to-version OpenAPI refactor work on an existing `.tst`.
- Use Skill 060 instead when the user already wants exactly one constrained REST Client subtree analyzed or refactored.
- This card is not a general owner for ordinary rename, enable/disable, copy, move, delete, or routine configuration edits on existing assets when dedicated API-backed owners exist.
- This is a hybrid card: local file/YAML evidence owns source asset identity, copied-target structure, and read-only analysis, while runtime ids and API-normalized write branches stay behind Skill 050.
