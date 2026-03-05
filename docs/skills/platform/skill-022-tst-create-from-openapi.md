# Skill 022: Create TST from OpenAPI

## 1) Skill Name
Create a new SOAtest `.tst` from OpenAPI/Swagger.

## 2) Objective
Create a brand-new `.tst` whose initial tests are generated from OpenAPI/Swagger using `POST /v6/files/tsts/swagger`.

## 3) Scope
- In scope:
  - create `.tst` from OpenAPI/Swagger URL or file-system path
  - verify resulting test file and root suite creation
  - optional environment creation flag
  - optional cleanup via file delete
- Out of scope:
  - traffic-based generation
  - post-generation test customization
  - WSDL/RAML generation (separate endpoints)

## 4) Inputs
- Required:
  - parent directory id (for example `/TestAssets`)
  - test file base name
- Conditional required (must be resolved before create):
  - service-definition source location (user-provided URL or file-system path)
- Optional:
  - `createEnvironment` object

## 4.1) Required Input Resolution Rule
- Resolve service-definition input using this precedence:
  1. explicit value in current user request,
  2. value confirmed earlier in current session,
  3. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
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
1. Confirm source location mode:
  - URL mode (user-provided URL)
  - File-path mode (user-provided path, internally normalized to API location fields)
2. For URL mode, verify reachability from server runtime (not only local browser):
  - probe the exact URL and expect `200`
3. Verify format/source alignment:
  - endpoint expects OpenAPI/Swagger content for this skill
4. Verify parent target:
  - `parent.id` exists and is writable (for example `/TestAssets`)
5. If preflight fails, stop before create and correct source/parent first.

## 5.2) Name Collision Gate (Fallback)
Run this guard only when target name uncertainty exists (for example after ambiguous write output) or when create returns `already exists`:
1. Query existing `.tst` files under target parent:
  - `GET /v6/descendants/files?id=/TestAssets&type=tst`
2. Check exact file-name match for `<name>.tst`.
3. If match exists, do not retry create blindly. Branch by user intent:
  - reuse existing file,
  - delete-and-recreate,
  - choose a new name.

## 6) Procedure
1. Resolve required service-definition location from user-provided URL or file path using the input-resolution rule.
2. If location is not confidently resolved, ask user for OpenAPI/Swagger source and stop before write.
3. Build create payload:
   - `parent.id=<directory-id>`
   - `name=<file-base-name>`
   - `location.url=<openapi-url>` or `location.id=<workspace-file-id>`
4. Create file from service definition:
   - `POST /v6/files/tsts/swagger`
5. Verify response:
   - response `id` ends with `.tst`
   - response includes root `Test Suite` in `relationships.childrenRel`
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
- Applies to SOAtest: Yes (validated for OpenAPI endpoint).
- Related endpoints:
  - `POST /v6/files/tsts/swagger`
  - `POST /v6/files/tsts/wsdl`
  - `POST /v6/files/tsts/raml`
- API-first rule:
  - Build payload from `tstsSwaggerRequest` schema directly; do not require pre-existing generated files as templates.

## 12) Current Validation Status (2026-03-03)
- Validated create and cleanup in this workspace using URL location.
- Evidence artifacts:
  - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22_create_swagger_tst_request.json`
  - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22_create_swagger_tst_response.json`
  - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22_delete_swagger_tst_response.json`

- Additional live validation using OpenAPI YAML URL:
  - source: `http://localhost:8090/parabank/services/bank/openapi.yaml`
  - evidence artifacts:
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22b_create_swagger_yaml_tst_request.json`
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22b_create_swagger_yaml_tst_response.json`
    - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/skill22b_delete_swagger_yaml_tst_response.json`
