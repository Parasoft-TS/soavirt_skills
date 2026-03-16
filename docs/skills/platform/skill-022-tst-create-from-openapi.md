# Skill 022: Create TST from OpenAPI

## 1) Skill Name
Create a new SOAtest `.tst` from OpenAPI/Swagger.

## 2) Objective
Create a brand-new `.tst` whose initial tests are generated from OpenAPI/Swagger using `POST /v6/files/tsts/swagger`. In this repo's broad-authoring model, that generated REST Client family is the standard unconstrained / None-mode branch, and downstream generated-client request/auth mutation routes through Skill 020 rather than Skill 059. Caller-owned post-generation template normalization, such as adding `BASEURL` and rewriting generated client hosts to `${BASEURL}` plus relative path/query, remains outside this card.

## 3) Scope
- In scope:
  - create `.tst` from OpenAPI/Swagger URL or file-system path
  - verify resulting test file and root suite creation
  - deterministic handling of the optional environment-creation request
  - canonical identification of the generated REST Client family as the standard unconstrained / None-mode branch used by broad authoring in this repo
  - optional cleanup via file delete
- Out of scope:
  - traffic-based generation
  - post-generation test customization beyond identifying the generated client family for downstream handoff
  - caller-owned post-generation template normalization such as adding `BASEURL` to the generated local environment or rewriting generated REST Client hosts to `${BASEURL}` plus relative path/query
  - WSDL/RAML generation (separate endpoints)

## 3.2) Routing Boundary (Required)
- Use this card as a direct route only when the user explicitly requests creating a **new `.tst`** from an OpenAPI/Swagger source.
- Do not use this card as the first route for help-style or outcome-level prompts (for example, `help me create tests for this OpenAPI`) when output mode is not explicit.
- For those ambiguous prompts, route to `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` first.

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md` (post-create traceability/tagging handoff)
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`

## 4) Inputs
- Required:
  - parent directory id (default to the active project root when a project is already active and the user did not override it; for example `/TestAssets/<Project>` or `/TestAssets`)
  - test file base name
- Conditional required (must be resolved before create):
  - service-definition source location (user-provided URL or file-system path)
- Optional:
  - `restCreateEnvironment` object, but only when the environment branch was already explicitly resolved using Skill 064 generation modes (`disabled`, `local_managed`, `reference_external`)

## 4.1) Required Input Resolution Rule
- Resolve service-definition input using this precedence:
  1. explicit value in current user request,
  2. active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
  3. value confirmed earlier in current session,
  4. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
- When a project is already active and `parent.id` was not explicitly chosen, default the new `.tst` to that project's root test-assets directory rather than a repo-global catch-all folder.
- Apply confidence gate before create:
  - proceed only when one candidate is unambiguous and format-compatible with OpenAPI/Swagger,
  - if source is missing, conflicting, or weakly inferred, ask a targeted user question and wait for confirmation,
  - do not call `POST /v6/files/tsts/swagger` without a confirmed source location.

## 5) Preconditions
- API reachable and authenticated.
- parent directory id exists and is writable.
- service definition location is reachable/readable by server runtime.

## 5.1) Service-Definition Source Readiness Preflight
Run this preflight before `POST /v6/files/tsts/swagger`:
1. Apply Skill 050 Profile B for `.tst` generation create/readback compatibility.
2. Confirm source location mode:
  - URL mode (user-provided URL)
  - file-path mode (user-provided path, internally normalized to API location fields)
3. For URL mode, verify reachability from server runtime (not only local browser):
  - probe the exact URL and expect `200`
4. Verify format/source alignment:
  - endpoint expects OpenAPI/Swagger content for this skill.
5. If preflight fails, stop before create and correct source/parent first.

## 5.2) Name Collision Gate (Fallback)
Run this guard only when target name uncertainty exists (for example after ambiguous write output) or when create returns `already exists`:
1. Query existing `.tst` files under target parent:
  - `GET /v6/descendants/files?id=<resolved-parent-id>&type=tst`
2. Check exact file-name match for `<name>.tst`.
3. If match exists, do not retry create blindly. Branch by user intent:
  - reuse existing file,
  - delete-and-recreate,
  - choose a new name.

## 6) Procedure
1. Resolve required service-definition location from user-provided URL or file path using the input-resolution rule.
2. If location is not confidently resolved, ask user for OpenAPI/Swagger source and stop before write.
3. Build create payload using the resolved project-aware parent and source location:
   - `parent.id=<directory-id>`
   - `name=<file-base-name>`
   - `location.url=<openapi-url>` or `location.id=<workspace-file-id>`
   - default v1 branch: omit `restCreateEnvironment`, rely on the generator's built-in local environment behavior, and do not plan a separate post-create environment creation step
   - include `restCreateEnvironment` only when the current request or already-confirmed upstream context explicitly resolves a non-default or explicitly parameterized environment branch from Skill 064
4. Create file from service definition:
   - `POST /v6/files/tsts/swagger`
5. Verify response:
   - response `id` ends with `.tst`
   - response includes root `Test Suite` in `relationships.childrenRel`
   - inspect the generated `.tst` environment state and reuse it as the initial environment context rather than creating a second environment by default
5.1 If create output is ambiguous (truncated/missing fields), reconcile before retry:
  - run a deterministic read check for exact `<name>.tst` under parent,
  - if found, treat create as successful and continue,
  - retry create only when reconciliation confirms file is absent.
6. Optional cleanup:
   - `DELETE /v6/files?id=<created-tst-id>`

## 7) Canonical Payload (API-First)
```json
{
  "parent": {
    "id": "/TestAssets"
  },
  "name": "My_OpenAPI_Test",
  "location": {
    "url": "http://localhost:8090/parabank/services/bank/openapi.json"
  }
}
```

## 8) Validation
- Expected HTTP status codes:
  - `200` on successful create
  - `400` malformed payload
  - `404` invalid parent or unreachable/invalid location reference
- Post-condition checks:
  - returned id/url are present
  - generated `.tst` contains root `Test Suite`

## 9) Failure Modes
- `400 Invalid payload` for malformed JSON or schema mismatch.
- `404` when `parent.id` is invalid.
- Service-definition retrieval/parsing errors if URL is unreachable, protected, or invalid OpenAPI/Swagger.
- Source readiness mismatch: URL reachable from client but not from server runtime.
- Missing or ambiguous service-definition location when write is attempted.
- Blind retry after ambiguous output can create name-collision churn (`already exists`) and hide first-write success.

## 10) Safety / Rollback
- Write operation on workspace filesystem.
- Rollback: delete created `.tst` via `DELETE /v6/files?id=...`.

## 11) Reuse Notes
- Primary target: SOAtest.
- Related endpoints:
  - `POST /v6/files/tsts/swagger`
  - `POST /v6/files/tsts/wsdl`
  - `POST /v6/files/tsts/raml`
- API-first rule:
  - Build payload from `tstsSwaggerRequest` schema directly; do not require pre-existing generated files as templates.
- If requirements traceability or tagging is requested, run Skill 009 on the root test suite immediately after creation.
- Use Skill 064 for `restCreateEnvironment` selection and any follow-on environment mechanics; v1 default generation mode is `local_managed`, which for brand-new `.tst` generation means relying on the generator's built-in local environment behavior unless the user or already-confirmed upstream context explicitly selects another supported branch.
- When invoked directly from routing, this card may be the full workflow for generation-only intent.
- In this repo's generation model, the REST Clients created by this OpenAPI generator are the standard unconstrained / None-mode generated branch; do not reinterpret Skill 022 broad generation as creating constrained REST Clients by default.
- When invoked from `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` or `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md` during broad authoring, this card owns only initial `.tst` generation plus create/readback verification.
- In that caller context, downstream generated-client request/auth mutation and readiness work remain with Skill 020 plus the caller's later orchestration phases; readiness remediation, negative/security authoring, calibration, validation enrichment, and any post-generation `BASEURL` normalization remain owned by the caller and its downstream orchestration/leaf skills.
