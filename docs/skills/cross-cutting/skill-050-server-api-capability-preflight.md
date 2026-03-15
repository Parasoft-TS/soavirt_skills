# Skill 050: Server API Capability Preflight
## 1) Skill Name
Server API capability preflight (API-branch safety, compatibility, and fallback routing)
## 2) Objective
Prevent trial-and-error mistakes when a branch interacts with the localhost API by standardizing API-root normalization, capability classification, `404` disambiguation, fallback routing, and opportunistic transient capability reuse.
## 3) Scope
- In scope:
  - API-read safety for runtime-object discovery and other API-mediated reads
  - API-write/execution safety before semantic mutation, generation, execution, or diagnostics
  - base URL / API-root normalization
  - branch-category capability classification
  - `404` disambiguation
  - fallback-routing policy by branch class
  - transient capability-ledger reuse
- Out of scope:
  - pure local filesystem operations
  - local file-backed read-only analysis
  - endpoint-specific payload schemas and detailed request/response choreography owned by other skills
## 3.1) Canonical Ownership Boundaries
This card owns:
- branch-category policy for API interaction
- base-URL/path normalization
- `404` disambiguation
- capability classification
- capability cache/ledger rules
- gate conditions for allowing an API branch to continue
This card does not own:
- endpoint payload schemas
- object semantics
- leaf-skill request/response details
## 4) Inputs
- Required:
  - resolved or candidate `baseUrl`
  - intended API branch class
- Optional:
  - force-refresh flag
  - transient capability cache key/path
## 5) Preconditions
- The branch actually needs API interaction.
- Authentication requirements are satisfied for the intended branch.
## 5.1) Capability Ledger (Optional, Transient)
- When available, read and persist capability outcomes in `work/cache/capability-ledger/<baseurl-key>.json`.
- Treat the ledger as an opportunistic transient cache only.
- Cache absence, deletion, unreadability, or write failure must never block live preflight.
- Local-only branches do not consult or update the capability ledger.
## 6) Procedure
### Lane A: API-read preflight
Use this lane for API-mediated read-only branches such as runtime-object discovery.
1. Confirm the usable API root with a lightweight read probe when no valid cached root result exists.
2. Consult the transient capability cache opportunistically when present.
3. Apply only the minimal profile checks needed for the read branch.
4. If id/path ambiguity remains unresolved, stop and ask rather than guessing.
### Lane B: API-write / execution preflight
Use this lane before API generation, semantic mutation, execution, or diagnostics.
1. Confirm the usable API root with a lightweight read probe when needed.
2. Consult the transient capability cache opportunistically when present.
3. Apply the matching write/execution profile.
4. Classify the route before continuation.
5. If the route is unsupported or ambiguous, stop and hand off to the documented fallback route.
## 6.1) Branch Profiles
- Profile A: API-read / runtime-object discovery
  - use when the branch needs API-authoritative runtime ids or other API-mediated reads
- Profile B: API generation / create / readback
  - use for API-based asset generation and create flows
- Profile C: semantic mutation
  - use for API PUT/POST/PATCH/DELETE branches that change runtime objects or semantic asset structures
- Profile D: execution / traffic / diagnostics
  - use for `/testExecutions`, traffic retrieval, and related diagnostics branches
- Profile E: explicit transfer / sync fallback
  - keep only if a meaningful transfer/sync fallback still survives after the platform-family rewrite
## 6.2) Capability Classification and `404` Disambiguation
- `200`: supported and successful
- `400`: path/method likely supported; classify as payload/input issue unless stronger evidence contradicts it
- `401` / `403`: auth/license blocked; stop the branch and resolve access
- `405`: method unsupported for that endpoint; route through the documented fallback for the branch profile
- `404`: ambiguous until disambiguated
`404` disambiguation sequence:
1. Re-check the active base path and target id/path assumptions.
2. Verify endpoint-path/method presence from cached OpenAPI when available.
3. Classify outcome:
  - input/id/path mismatch -> fix inputs and stay in the same branch class
  - unsupported/unavailable route -> hand off to the documented fallback route for the profile
## 6.3) Capability Ledger Reuse
For each cached result, store only transient compatibility evidence such as:
- timestamp
- base URL key
- branch profile
- endpoint + method
- final classification
- selected fallback (if any)
Reuse rules:
- do not re-probe unchanged endpoint-method tuples within compatible API branches when a valid cached result already exists
- do not treat cached results as durable repository knowledge
## 7) Canonical Fallback Routing References
- runtime-object discovery and targeting discipline:
  - `docs/workflow/agent-workflow.md`
  - `docs/skills/platform/skill-001-shared-introspection.md` when the branch starts local-first and later needs API runtime ids
- execution payload/traffic details:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- API generation / mutation details:
  - owning atomic or composite skill for that branch
## 8) Validation
- Lane A succeeds only when the API read branch can proceed without base-path or id/path guessing.
- Lane B succeeds only when the write/execution route is classified and the branch can continue without trial-and-error retries.
- Cache reuse never becomes a hard dependency.
## 9) Failure Modes
- Skill 050 is skipped for a branch that actually uses the API.
- Skill 050 is invoked for a local-only branch where it is not needed.
- A `404` is treated as endpoint incompatibility before checking base path or target assumptions.
- Cached transient results are treated as durable truth.
- Profile definitions duplicate leaf-skill payload behavior instead of staying at the policy/gating level.
## 10) Applicability Examples
Use Skill 050 for:
- resolving a suite/tool/datasource id through the localhost API
- generating a new `.tst` through API-based creation
- updating a runtime object through an API mutation path
- running executions or traffic diagnostics
Do not use Skill 050 for:
- renaming a local `.tst` file under `TestAssets/`
- deleting a local project resource file
- analyzing a `.tst` by local path using YAML/file evidence only
Hybrid rule:
- start local-first, and enter Skill 050 only at the moment an API interaction becomes necessary
## 11) Reuse Notes
- Keep this card as the canonical owner of API-access discipline.
- Keep endpoint-specific semantics in the owning leaf skills.
