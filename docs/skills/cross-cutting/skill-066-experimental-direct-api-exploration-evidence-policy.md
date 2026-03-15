# Skill 066: Experimental Direct API Exploration Evidence Policy

## 1) Skill Name
Bounded direct API exploration and transient evidence policy for the explicit opt-in experimental broad-authoring lane.

## 2) Objective
Define how the experimental lane performs direct endpoint exploration before SOAtest authoring: how request hypotheses are built, how method classes are probed, how retries and reconciliation are bounded, what evidence is considered legitimate, how transient exploration artifacts are stored, and how successful findings are normalized into mutation-ready request bundles for downstream SOAtest authoring.

## 3) Scope
- In scope:
  - direct HTTP exploration for REST/OpenAPI operations in the explicit opt-in experimental lane
  - request-hypothesis generation from contract, project context, user values, and same-run evidence
  - method safety classes for observational, state-advancing, and destructive probes
  - bounded retry/attempt rules
  - CRUD-oriented reconciliation after state-changing probes
  - transient exploration-ledger shape and evidence legitimacy rules
  - normalized request-bundle production for downstream SOAtest mutation owners
  - outcome classification for explored operations
- Out of scope:
  - activating direct exploration as a default baseline outside the experimental lane
  - guessing credentials or inventing secrets
  - unbounded trial-and-error probing
  - replacing the lower-layer SOAtest mutation owners
  - durable storage of exploration-derived OpenAPI structure in project records

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-059-constrained-rest-client-yaml-fallback.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`

## 4) Inputs
- Required:
  - selected operation slice
  - OpenAPI contract details for the selected operations
  - active project context relevant to the branch
  - explicit user-provided request values or constraints when available
- Optional:
  - transient exploration preanalysis artifact for reproducibility
  - already-learned same-run identifiers or reusable successful values

## 5) Preconditions
- The branch is inside the explicit opt-in experimental lane.
- The target source is REST/OpenAPI.
- The API branch has passed capability preflight for the needed read/write calls.
- Credentials are either explicitly available or the operation can be classified `blocked-auth`; do not guess credentials.

## 6) Procedure
### 6.1 Inputs and Outputs
1. Treat the exploration phase outputs as:
  - one per-operation exploration dossier in a transient ledger under `work/`
  - a selected canonical happy-path request when one reaches `usable happy path`
  - response-shape and validation-design hints derived from the successful response
  - final blocked/exhausted/mismatch classifications when no usable happy path is found
  - one normalized mutation-ready request bundle per successful operation
2. Treat the exploration ledger as the evidence contract for later authoring and validation; downstream phases should consume it rather than rediscovering the same facts through unfinished SOAtest tests.

### 6.2 Deterministic Operation Ordering
3. Process the selected operations in deterministic order:
  - Class A: `GET`, `HEAD`, `OPTIONS`
  - Class B: `POST`, `PUT`, `PATCH`
  - Class C: `DELETE`
4. Within each class, preserve stable OpenAPI order.
5. Allow earlier successful operations to contribute reusable evidence only to later operations in the same run when the field meaning is clearly compatible.

### 6.3 Request-Hypothesis Generation
6. Before the first live probe for an operation, build one request hypothesis containing:
  - method and path
  - resolved base URL candidate
  - auth/header hypotheses
  - path/query inventory with candidate values and provenance
  - request-body schema summary when applicable
  - candidate payload object when applicable
  - expected response families from the contract
7. Candidate-value precedence is:
  1. explicit user-provided value
  2. approved project-context value or durable project data/reference
  3. operation-local OpenAPI examples/defaults/enums
  4. reusable values learned from earlier successful exploration in the same run when meaning is clearly compatible
  5. synthesized schema-conformant placeholder values
8. Never synthesize credentials, secrets, or auth tokens.
9. If auth remains unresolved, classify the operation `blocked-auth` rather than probing blindly.

### 6.4 Method Safety Classes and Probing Stance
10. Treat `GET`, `HEAD`, and `OPTIONS` as observational probes once base URL and auth are sufficiently resolved.
11. Treat `POST`, `PUT`, and `PATCH` as state-advancing probes:
  - probing is allowed when the selected operation is in scope and the request structure is meaningful enough to attempt
  - incidental state change is accepted as normal exploration evidence, not a default failure condition
  - every successful or partially successful state-changing probe must be followed by reconciliation
12. Treat `DELETE` as destructive probing with stricter target-resolution requirements:
  - probing is allowed only when target identity is concrete enough to avoid blind destructive guessing
  - post-delete reconciliation is mandatory whenever the contract/runtime makes it possible

### 6.5 Bounded Attempt Loop
13. For each operation, use a bounded attempt loop:
  - Attempt 0: hypothesis assembly only
  - Attempt 1: best initial request from the current evidence ladder
  - Attempt 2: allowed only when Attempt 1 produced narrowing evidence
  - Attempt 3: allowed only when Attempt 2 produced fresh narrowing evidence
  - no fourth attempt
14. Blind retries are forbidden.
15. Each retry must cite the new evidence that changed the request.

### 6.6 Reconciliation After State Change
16. After any successful or state-changing write probe, run reconciliation whenever the contract/runtime makes it possible.
17. Preferred reconciliation order:
  1. direct `GET` of the returned or inferred resource id/path
  2. collection/list read that can confirm the new current state
  3. another contract-compatible existence/update/delete confirmation call
18. Record both the immediate mutation response and the reconciled post-state in the same attempt record.
19. For `POST`, treat the immediate create response as the primary authored `POST` baseline unless it is too sparse to support meaningful downstream design.

### 6.7 Outcome Classification
20. Each explored operation ends in exactly one primary state:
  - `usable happy path`
  - `blocked-auth`
  - `blocked-input`
  - `probe-exhausted`
  - `negative-only-evidence`
  - `spec-runtime-mismatch`
21. State change is not a primary state by itself; it is an attempt annotation paired with reconciled post-state.
22. When several attempts succeed, choose the canonical happy-path request by:
  1. highest-precedence input provenance
  2. simplest successful request shape
  3. response and reconciled post-state that are semantically complete enough for validation design

### 6.8 CRUD-Oriented Chaining Across Operations
23. Allow same-run evidence chaining across CRUD-related operations in the selected slice:
  - `POST` may establish resource ids for later `GET`/`PUT`/`PATCH`/`DELETE`
  - reconciliation `GET` responses may provide better state-shape hints for later operations
  - list/read operations may supply ids or field values needed for later updates or deletes
24. Prefer list/read-first when the contract exposes cheap deterministic discovery.
25. Prefer create-first when creating a fresh resource is the cleanest way to establish controlled state for the rest of the slice.

### 6.9 Exploration Ledger Contract
26. The transient ledger under `work/` should record per operation:
  - operation identity
  - method safety class
  - assumptions used for base URL/auth/headers
  - every attempt with request, provenance, response status, media type, and short interpretation
  - whether the attempt observed or changed state
  - reconciliation details for write attempts
  - newly learned ids or reusable values
  - the canonical happy-path request when one exists
  - response-shape summary
  - candidate volatile fields
  - candidate negative families
  - final classification and blocker reason when not successful
27. Keep these artifacts under `work/`; they are supplementary transient evidence, not reusable canonical project context.

### 6.10 Evidence Legitimacy Rule
28. Exploration evidence is legitimate for downstream authoring or validation only when:
  - the explored operation was actually in the selected slice
  - the evidence comes from the canonical request/response pair chosen for the authored branch, not from an abandoned alternate attempt
  - payload/body classification is based on the observed semantic response body first, with headers only as supporting evidence
  - the evidence is not merely a pre-remediation or underconfigured failure symptom
29. If legitimacy is weak or ambiguous, stop the slice as blocked/uncertain rather than laundering weak evidence into later phases.

### 6.11 Mapping into SOAtest Authoring
30. Produce one normalized request bundle per successful operation containing:
  - endpoint/base URL target
  - method
  - headers except tool-owned `Content-Type`
  - path/query values
  - body payload and content type when applicable
  - expected happy-path status-code set
31. Downstream mutation should route through the existing stable owners:
  - Skill 020 for unconstrained/None-mode REST Clients
  - Skill 059 for constrained same-operation clients

## 7) Validation
- Request hypotheses are built before live probes begin.
- Candidate-value precedence remains deterministic.
- No credential guessing occurs.
- Blind retries are forbidden.
- Write probes are reconciled whenever possible.
- Ledger artifacts stay transient under `work/`.
- Exploration evidence is normalized into mutation-ready bundles rather than left as ad hoc notes.
- Only legitimate exploration evidence is allowed to influence downstream authoring and validation.

## 8) Failure Modes
- Same-run exploration values are reused across incompatible fields.
- Direct probes proceed even though auth is unresolved.
- State-changing methods are treated as fail-open with no reconciliation.
- `DELETE` probes proceed without strong target identity.
- The retry budget is exceeded or used for blind guesses.
- The ledger records status codes but not enough request/provenance context to explain what succeeded.
- Exploration-derived OpenAPI structure is persisted durably into project context.
- Downstream phases author from a non-canonical exploration attempt.

## 9) Safety / Rollback
- This policy authorizes bounded direct exploration, including some state-changing requests, only inside the explicit experimental lane.
- The main safety controls are:
  - strict opt-in lane boundary
  - no credential guessing
  - bounded attempts
  - reconciliation after write probes
  - transient evidence storage under `work/`
- Rollback of authored SOAtest assets remains the responsibility of the downstream mutation owners.

## 10) Reuse Notes
- This card is a cross-cutting policy/evidence owner, not a first-choice operator-facing route.
- Skill 065 is the expected caller.
- Skill 067 consumes the legitimacy and evidence rules defined here.
- Keep stable execution-backed evidence rules unchanged outside this explicit lane.
