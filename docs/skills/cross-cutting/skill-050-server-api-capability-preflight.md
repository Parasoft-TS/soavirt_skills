# Skill 050: Server API Capability Preflight

## 1) Skill Name
Server API capability preflight (endpoint-method compatibility and fallback routing)

## 2) Objective
Minimize trial-and-error during server interactions by standardizing per-session API capability classification, fallback routing, and reusable evidence for server-API-mediated runtime branches, with fail-closed gating before writes.

## 3) Scope
- In scope:
  - base URL/API-root reachability probe.
  - operation-class/runtime-branch preflight profiles.
  - capability-sensitive read, write, transfer, and execution branch compatibility checks.
  - endpoint-method capability classification and `404` disambiguation.
  - deterministic fallback routing for unsupported/unavailable paths.
  - per-session capability ledger recording for reuse.
- Out of scope:
  - modifying OpenAPI cache files.
  - replacing endpoint-level behavior in atomic skills.
  - redefining payload schemas owned by endpoint-specific cards.

## 3.1) Canonical Ownership Boundaries
- Endpoint role semantics for discovery/targeting are owned by:
  - `docs/skills/platform/skill-001-shared-introspection.md`
- File transfer safety and encoding invariants are owned by:
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
- Execution payload and traffic retrieval constraints are owned by:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- This skill owns runtime API capability preflight policy only (classification, routing, and evidence model).

## 4) Inputs
- Required:
  - resolved `baseUrl`.
  - intended workflow branch.
- Optional:
  - force-refresh flag for capability re-probe.
  - cached OpenAPI spec key/path.

## 5) Preconditions
- Server is reachable on network.
- Authentication requirements are satisfied.
## 5.1) Preflight Evidence Artifact (Required)
- Persist and read capability outcomes from `work/cache/capability-ledger/<baseurl-key>.json`.
- Ledger lookup is mandatory before any runtime capability probe.
- Reuse existing verified results for unchanged cache keys to avoid redundant probing across:
  - multi-skill composition for one operator request,
  - multiple operator prompts in the same ongoing agent session.

## 5.2) Canonical Cache Key
- Cache key tuple:
  - `baseUrlKey + endpointPath + method + operationProfile + authContextKey`
- `authContextKey` should distinguish principal/auth mode at minimum (for example user or auth scheme identity marker).

## 6) Procedure
1. Determine operation class for the pending branch (see Section 6.1 profiles).
2. Load capability ledger from `work/cache/capability-ledger/<baseurl-key>.json` (create if missing).
3. Resolve cache keys for planned endpoint-method checks.
4. For each planned check:
  - if a valid cache entry exists and no invalidation condition applies (Section 6.4), reuse cached classification and skip runtime probe.
  - otherwise queue the check for runtime probing.
5. Probe base API reachability with a lightweight read when no valid cached reachability record exists:
   - `GET /v6/children`
6. Perform spec-first verification for queued checks:
  - when cached OpenAPI is available, confirm endpoint-path/method presence before runtime probe.
7. Apply the matching preflight profile from Section 6.1.
8. Classify each new runtime probe result using Section 6.2.
9. Update ledger entries for new/invalidated checks and persist ledger file.
10. Select fallback route only from the matched profile and documented references in Section 7.
11. Continue execution using only profile-verified endpoint-method pairs.

### 6.1) Canonical Runtime Branch Profiles
- Profile A: filesystem and asset target resolution (read-only)
  - use endpoint-role semantics from Skill 001:
    - `GET /v6/children`
    - `GET /v6/descendants/files`
    - `GET /v6/descendants/assets` only with asset/file ids (never directory ids).
- Profile B: `.tst` generation/create-readback branches
  - pre-write checks:
    - parent id resolvable/writable.
    - name-collision check when name uncertainty exists (or after ambiguous output).
  - write path:
    - `POST /v6/files/tsts*` endpoint selected by target generation card.
  - readback route:
    - primary: `GET /v6/children?id=<tst-id>`
    - optional context: `GET /v6/descendants/assets?id=<tst-id>`
  - compatibility guard:
    - do not use `GET /v6/files/tsts?id=...` as a cross-server readback assumption.
- Profile C: file transfer branches
  - resolve id existence with read probes before upload/download.
  - enforce UTF-8 without BOM for upload-bound content per Skill 002.
  - require re-download verification after upload writes.
- Profile D: execution and traffic-observation branches
  - payload contract and fail-fast behavior are owned by Skill 012.
  - traffic retrieval must use two-step suite/viewer flow:
    - suite `entityId` -> returned viewer ids -> focused viewer payload call.
- Profile E: suite/object/tool mutation branches
  - resolve target ids/parents through current readback before write.
  - use read-merge-write policies:
    - tools: Skill 049
    - non-tool objects: Skill 053
  - avoid synthetic sparse-write probes as method compatibility checks.

### 6.2) Capability Classification and `404` Disambiguation
- `200`: supported and successful.
- `400`: method/path likely supported; classify payload/input issue unless contradictory evidence exists.
- `401`/`403`: auth/license blocked; stop write branch and resolve access.
- `405`: method unsupported for endpoint; route via profile fallback.
- `404`: ambiguous until disambiguated.

`404` disambiguation sequence:
1. Re-verify referenced ids/parents via read endpoints (target-resolution check).
2. Verify endpoint-path/method presence from cached OpenAPI when available.
3. Classify outcome:
  - input/id mismatch -> fix inputs; keep same branch path.
  - endpoint unavailable/unsupported in target runtime -> route to profile fallback.

### 6.3) Capability Ledger Format (Canonical)
For each probed endpoint-method pair, store:
- timestamp.
- base URL key.
- operation class/profile.
- endpoint + method.
- probe intent.
- HTTP status.
- final classification.
- selected fallback (if any).
- evidence reference (artifact path or response id).
- cache key tuple fields from Section 5.2.

Reuse rules (mandatory):
- do not re-probe unchanged endpoint-method tuples within the same session when ledger contains valid verified outcomes.
- always persist ledger updates after preflight completion so subsequent prompts in the same session can reuse them.

### 6.4) Cache Invalidation Rules (Mandatory)
Invalidate and re-probe a cached entry when any of the following is true:
1. `baseUrl` changes.
2. auth context changes (principal/scheme marker differs from `authContextKey`).
3. force-refresh is requested.
4. runtime behavior contradicts cached classification (for example cached supported path later returns `405`/contradictory `404` after id/path validation).

## 7) Canonical Fallback Routing References
- Discovery/asset-graph endpoint-role selection:
  - `docs/skills/platform/skill-001-shared-introspection.md`
- File transfer encoding/round-trip safety:
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
- Execution payload schema + traffic retrieval flow:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- `.tst` create/readback branch behavior:
  - `docs/skills/platform/skill-021-tst-create-empty.md`
  - `docs/skills/platform/skill-022-tst-create-from-openapi.md`
  - `docs/skills/platform/skill-023-tst-create-from-wsdl.md`
  - `docs/skills/platform/skill-024-tst-create-from-raml.md`
  - `docs/skills/platform/skill-025-tst-create-from-xsd.md`

## 8) Validation
- Preflight output includes ledger entries for all probed endpoint-method pairs.
- No runtime branch continues with an endpoint-method pair classified unsupported/unavailable once the matched profile has selected a supported route; write branches remain fail-closed.
- `404` outcomes are annotated as either:
  - input/id mismatch (correct-and-retry same endpoint), or
  - confirmed unavailable endpoint/path (fallback route).

## 9) Failure Modes
- Preflight omitted on a capability-sensitive branch, causing mid-run `405`/`404` retries.
- Over-broad probing increases startup latency and reintroduces trial behavior.
- Misinterpreting auth failures (`401`/`403`) as method incompatibility.
- Misclassifying write-probe `404` as endpoint incompatibility without id/path disambiguation.
- Treating payload-shape `400` as unsupported method without corroborating evidence.
- Duplicating canonical endpoint behavior in downstream cards, creating policy drift.

## 10) Reuse Notes
- Shared across SOAtest and Virtualize when server-API capability differences can affect runtime routing.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Use as a dependency for composite orchestration, runtime capability-sensitive read/transfer branches, and write-capable atomic cards.
- Keep this skill as the canonical preflight policy surface; keep endpoint/payload specifics in their atomic owner cards.
