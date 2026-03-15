# Skill 059: Single Constrained REST Client Lifecycle (YAML Promotion + API Body Normalization)
## 1) Skill Name
Create/read/update/copy/delete one service-definition-backed constrained REST Client, using local-file YAML promotion for constrained binding and REST Client API `GET/PUT/GET` normalization for constrained JSON request bodies.
## 2) Objective
Provide one reliable v1 owner for single constrained REST Client lifecycle work in the merged workspace. This card supports:
- creating a fresh constrained REST Client from scratch
- reading, copying, deleting, and same-client updating one existing constrained REST Client
- same-operation request-value edits on an existing constrained client
- JSON body-bearing create/update flows only through the validated two-phase path:
  - minimal local YAML promotion into constrained mode
  - REST Client API body normalization through read-merge-write `PUT`
Operation identity is defined by OpenAPI path plus HTTP method, not by `operationId`.
## 3) Scope
- In scope:
  - one REST Client whose API readback indicates service-definition backing and whose persisted local YAML indicates constrained mode
  - local-file-backed `.tst` targeting plus runtime-object/API branches where needed
  - create a fresh constrained REST Client by:
    - creating or reusing a plain REST Client shell
    - promoting it through isolated local YAML edits inside one target tool block
    - optionally normalizing a JSON request body through REST Client API `GET/PUT/GET`
  - update of one existing constrained REST Client when the work stays within the same selected operation
  - copy via `POST /v6/tools/copy`
  - delete via `DELETE /v6/tools?id=...`
  - same-operation edits to constrained request/config surfaces for one target tool
- Out of scope:
  - broader multi-client refactor or migration orchestration
  - operation retargeting outside the validated shell/promotion workflow in this card
  - structural YAML edits outside one isolated target tool block
  - autonomous invention of new environment-variable names
  - non-JSON constrained payload modes
## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` when runtime execution/traffic verification is explicitly in scope
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
## 4) Inputs
- Required:
  - target `.tst` local path or enough file-backed identity to resolve it through Skill 001
  - target parent suite reference when creating or copying
  - target constrained REST Client runtime id or unique local tool path/name for read/update/delete
  - OpenAPI/schema location and base URL source for fresh constrained creation when they cannot be reused from the active project record or a same-spec peer
  - selected OpenAPI path plus HTTP method
- Optional:
  - REST Client/test name
  - caller-supplied variable tokens for base URL or schema URL
  - intended path/query/body request values
  - focused verification test name when runtime verification is explicitly in scope
## 5) Preconditions
- The local `.tst` is readable.
- For update branches, API readback plus local YAML confirm the target is already a constrained REST Client.
- For create/copy/delete/API-body branches, Skill 050 confirms the needed API route before mutation.
## 6) Procedure
0. Resolve the branch before mutation:
   - read existing constrained client
   - create a fresh constrained client
   - update one existing constrained client without changing the selected operation
   - copy one constrained client
   - delete one constrained client
1. Resolve the local `.tst` path first through Skill 001.
2. Use active project references/environment files plus optional same-spec local peers to determine schema/base-url style before editing. Do not invent new environment-variable names when project context already defines the owning source.
3. Enter Skill 050 only when the branch actually needs API interaction:
   - runtime tool resolution
   - create/copy/delete routes
   - REST Client `GET/PUT/GET` body normalization
4. Read branch:
   - use REST Client API readback plus local YAML evidence to confirm constrained/service-definition-backed state
5. Copy branch:
   - use `POST /v6/tools/copy`
   - verify the copied tool through API readback plus local file readback
6. Delete branch:
   - use `DELETE /v6/tools?id=...`
   - verify deletion through runtime readback plus local file readback
7. For create or YAML-surface update branches, invoke Skill 006 to establish rollback-preserving local YAML edit handling:
   - preserve the original local rollback source
   - isolate one target tool block before editing
8. Create/update the constrained YAML block with a local shell-first envelope:
   - for fresh creation, create or reuse a plain REST Client shell and promote it locally
   - keep the selected operation unchanged for same-operation edits
   - set or preserve the minimum proven constrained structures inside the isolated target block
   - keep edits block-local and avoid unrelated tool drift
9. For path/query/resource/config surfaces in the YAML branch:
   - path parameter edits stay within existing constrained path nodes
   - query parameter edits must keep all mirrored surfaces synchronized
   - semantically equivalent literalization is allowed only when the resolved value stays the same
10. For JSON body-bearing create/update branches, use the required second phase after local YAML promotion:
   - read the live tool with `GET /v6/tools/restClients?id=<tool-id>`
   - keep the full GET payload as the mutation base
   - derive the request payload shape from the selected operation's request schema/OpenAPI definition
   - source payload values under the Skill 058 ladder
   - write the full object back with `PUT /v6/tools/restClients?id=<tool-id>`
   - treat the REST Client PUT path as the authoritative normalizer for persisted constrained `formJson`
11. Save local YAML edits and confirm locally that only the intended target block changed.
12. Re-read the updated state and verify all required persisted-state evidence:
   - API readback still resolves the tool under the intended parent when an API branch was used
   - local YAML readback shows the intended constrained operation binding and request-value state
   - for JSON body-bearing branches, API readback shows `payload.inputMode = formJSON`
   - for JSON body-bearing branches, local YAML shows persisted `formJson` plus `MessagingClient_LiteralMessage`
13. Optional runtime verification branch:
   - run this only when the user explicitly asks for runtime verification or the owning workflow explicitly includes it
14. If required persisted-state verification fails:
   - restore the original local `.tst` through Skill 006 for YAML-based failure
   - restore the original full REST Client GET payload for API-body failure
## 7) Validation
- The target tool is present under the intended parent after create/update/copy.
- API readback shows service-definition backing and the intended operation binding when an API branch was used.
- Local YAML readback shows the intended constrained structures without unrelated drift.
- For constrained `Form JSON` bodies, API/YAML readback confirms the intended persisted request payload shape.
## 8) Failure Modes
- The branch treats local file-backed `.tst` identity as API-first instead of resolving the local path first.
- YAML promotion binds the wrong operation because the selected OpenAPI path/method were not confirmed first.
- Query edits drift because only one mirrored field was updated.
- Sparse/name-only REST Client PUT drops required configuration.
- Broad regex or whole-file replacement causes unrelated tool drift.
## 9) Safety / Rollback
- Always route local YAML rollback-preserving handling through Skill 006.
- Always keep the original full REST Client GET payload as the rollback source for API-body edits.
- Keep local before/after snapshots of the target tool block so non-target drift can be detected before finalizing the edit.
## 10) Reuse Notes
- This is a hybrid card: the `.tst` asset and YAML block are local-first, while runtime tool ids and constrained JSON body normalization remain API-mediated.
- Route broader multi-client refactor work to Skills 060 and 061.
- If the branch requires external environment-file attachment, referenced-environment node changes, or active-environment verification, delegate those mechanics to Skill 064 rather than folding them into local constrained-client edits.
