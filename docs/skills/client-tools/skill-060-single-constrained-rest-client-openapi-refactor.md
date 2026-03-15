# Skill 060: Single Constrained REST Client OpenAPI Refactor
## 1) Skill Name
Refactor one constrained REST Client subtree from one confirmed OpenAPI contract to another.
## 2) Objective
Refactor exactly one source-spec-bound constrained REST Client subtree from a confirmed source OpenAPI contract to a confirmed target OpenAPI contract, including one-client operation/parameter/request-body migration reasoning and supported downstream chained-tool repair, while using the local `.tst` path as the authoritative asset anchor and entering API branches only when runtime ids or API-normalized mutation are actually required.
This card is the lower-layer reusable leaf beneath broader bulk orchestration such as `Change Advisor`.
## 3) Scope
- In scope:
  - one constrained REST Client subtree only
  - one confirmed source OpenAPI -> one confirmed target OpenAPI migration slice
  - two modes:
    - `analysis`
    - `write`
  - operation mapping for the selected constrained REST Client
  - path/query parameter mapping
  - request-body field mapping
  - supported downstream chained-tool repair intent/application for:
    - JSON Validator
    - JSON Data Bank
    - JSON Assertor
    - Diff Tool
  - structured per-client result emission, client-slice structural verification, and rollback
- Out of scope:
  - multi-client inventory or grouped user review ownership
  - copied-file naming or whole-run sequencing
  - unconstrained REST migration
  - SOAP/WSDL migration
  - XML-family chained-tool refactoring
## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md`
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
  - mode: `analysis` or `write`
  - source `.tst` local path
  - stable client reference inside that file:
    - suite/scenario/test path context
    - target client identity within that path
  - source client YAML/config slice
  - immediate supported chained-tool descendants
  - confirmed source and target OpenAPI subsets needed for that client, from explicit inputs or active project references already established for the branch
- Conditionally required for `write` mode:
  - copied target `.tst` local path
  - exact copied client location inside that file
  - resolved client-local decisions
  - approved chained-tool repair contracts for any non-autonomous downstream changes
## 4.1) Evidence Boundary
- Analysis should reason from:
  - the local source `.tst` YAML slice for the target client subtree, and
  - normalized source/target OpenAPI structures for that same client slice
- Runtime/API reads are allowed only when a branch needs:
  - runtime tool identity
  - API-normalized constrained JSON body mutation
  - API readback verification of an approved API branch
- Do not use ad hoc runtime rediscovery of constrained-client internals as competing analysis evidence once the local YAML slice is established.
## 4.2) Result Contract
For `analysis` mode, return structured per-client results containing:
- client identity
- operation mapping result
- parameter mapping result
- request-body mapping result
- supported chained-tool repair intents
- per-domain decision states (`operationState`, `parameterState`, `requestBodyState`, `chainedToolState`)
- overall client decision state (`AUTO`, `REVIEW`, or `BLOCKED`)
- unresolved decision items and predicted write plan
For `write` mode, return:
- client identity
- applied edits in write order
- skipped or deferred edits
- client-slice structural verification result
- final client write state (`applied`, `partial`, `blocked`, or `rolled back`)
## 5) Preconditions
- The source `.tst` local path and target client local path context are resolved.
- The source and target OpenAPI inputs are resolved enough to build a one-client comparison.
- `write` mode must not begin while client-local unresolved decisions remain.
- `write` mode assumes the copied target `.tst` already exists and the original source `.tst` remains untouched.
## 6) Procedure
0. Respect orchestration ownership boundaries:
   - do not perform whole-run inventory here
   - do not own grouped user review here
1. Resolve mode: `analysis` or `write`.
2. Resolve the local `.tst` path and client local path context through Skill 001 if needed.
3. Build the one-client evidence bundle from the local YAML slice plus normalized source/target OpenAPI structures. When active project context exists, consult project references first so source/target contract anchors do not need to be rediscovered ad hoc.
### 6.1) Analysis Mode
4. Analyze operation mapping first:
   - path + HTTP method remain the primary key
   - treat `operationId`, summary text, and schema similarity as supporting evidence only
   - classify operation outcome as `safe-autonomous`, `proposal-plus-approval`, or `blocked`
5. Analyze path/query parameter mapping after operation context is known.
6. Analyze request-body mapping after operation context is known:
   - exact path+method continuity does not imply request-body compatibility
   - target-only fields must surface explicit candidate or omission proposals when defensible
   - required target-only fields with no defensible value source remain `blocked`
7. Analyze supported downstream chained tools using Skills 017 and 018 as canonical parent/output authority.
8. Emit unresolved decision items and the structured analysis result.
### 6.2) Write Mode
9. Require resolved client-local decisions before any mutation.
10. Lock the canonical copied target client location in the local copied `.tst` before the first write.
11. Apply exact in-scope source-spec and base-url edits locally in the copied client slice.
12. If the approved change includes constrained JSON body normalization or another API-only branch, enter Skill 050 and route that branch through Skill 059 on the resolved runtime client id.
13. Apply approved parameter, request-body, and supported chained-tool repairs in dependency order.
14. Perform client-slice structural verification:
   - confirm the copied client subtree remains structurally coherent in the local copied `.tst`
   - confirm updated downstream tool nodes remain resolvable
   - if an API-normalized body branch was used, confirm both local file readback and API readback match the approved persisted shape
15. If verification fails, roll back that client slice and return `rolled back`.
## 7) Validation
- `analysis` mode produces structured per-client results without mutating the source or copied `.tst`.
- For body-bearing operations, request-body mapping remains mandatory; path+method continuity alone is never sufficient for overall `AUTO`.
- `write` mode applies only approved/autonomous decisions and does not invent new required values.
- Client-slice structural verification is required before the slice is considered applied.
- When an API-normalized body branch is used, persisted body-shape verification must confirm the copied client reflects the approved target shape in both local file and API readback.
## 8) Failure Modes
- The branch treats runtime/API evidence as authoritative for file-backed client identity after the local `.tst` slice is already resolved.
- Exact path+method match collapses a body-bearing client to `AUTO` before explicit request-body analysis.
- Target-only body fields are silently dropped instead of surfacing as review items.
- Write completion is inferred from source-spec/base-url rewrites or generic file readability without confirming approved request-body shape when that domain changed.
## 9) Safety / Rollback
- This is a write-capable leaf in `write` mode and must remain fail-closed.
- Treat the client subtree as the rollback boundary when possible.
- If the client subtree cannot be left structurally coherent, restore that client slice and return `rolled back`.
## 10) Reuse Notes
- This is a hybrid card: local file/YAML evidence owns client-slice analysis and copied-target structure, while runtime ids and constrained JSON body normalization stay API-mediated through Skills 050 and 059.
- This card is not a general owner for ordinary rename, enable/disable, copy, move, delete, or routine configuration edits on existing assets when dedicated API-backed owners exist.
- Skill 059 owns constrained REST Client lifecycle mechanics; Skill 060 owns one-client source-to-target refactor reasoning plus supported downstream repair ownership.
