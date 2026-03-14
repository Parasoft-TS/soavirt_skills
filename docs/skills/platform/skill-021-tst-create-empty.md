# Skill 021: Create New Empty TST File

## 1) Skill Name
Create a new empty SOAtest `.tst` file via REST API.

## 2) Objective
Create a brand-new test asset with default root `Test Suite` using `POST /v6/files/tsts`.

## 3) Scope
- In scope:
  - create empty `.tst` (or reuse existing on name conflict)
  - verify created file id and default root suite
  - optional cleanup via file delete
- Out of scope:
  - auto-populating tests/tools
  - importing service definitions (handled by Skill 022)
  - creating from traffic

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md` (post-create traceability/tagging handoff)

## 4) Inputs
- Required:
  - parent directory id (for example `/TestAssets`)
  - test file base name (without `.tst` extension)

## 5) Preconditions
- API reachable and authenticated.
- parent directory id exists and is writable.

## 6) Procedure
1. Build create payload:
   - `parent.id=<directory-id>`
   - `name=<file-base-name>`
2. Create file:
   - `POST /v6/files/tsts`
3. If create returns name-conflict (`already exists`), switch to create-or-reuse mode:
  - resolve existing id by listing target folder descendants,
  - continue with the resolved existing `.tst` id.
4. Verify response/resolved id:
   - response `id` ends with `.tst`
5. Verify root suite with compatibility-safe reads:
  - apply Skill 050 Profile B readback route:
    - primary: `GET /v6/children?id=<tst-id>` and confirm `testSuite` child exists,
    - optional context: `GET /v6/descendants/assets?id=<tst-id>`.
6. Optional cleanup:
   - `DELETE /v6/files?id=<created-tst-id>`

## 7) Canonical Payload (API-First)
```json
{
  "parent": {
    "id": "/TestAssets"
  },
  "name": "My_New_Test"
}
```

## 8) Validation
- Expected HTTP status codes:
  - `200` on successful create
  - `400` malformed payload
  - `404` invalid/missing parent id
- Post-condition checks:
  - created or reused `id` is resolved
  - `GET /v6/children?id=<tst-id>` returns a default root `testSuite`

## 9) Failure Modes
- `400 Invalid payload: Message could not be parsed` when request JSON is malformed or BOM-encoded.
- `404` when `parent.id` is not a valid directory.
- Name conflict behavior may vary by server/workspace policy; treat `already exists` as recoverable with create-or-reuse flow.

## 10) Safety / Rollback
- Write operation on workspace filesystem.
- Rollback: delete created `.tst` with `DELETE /v6/files?id=...`.

## 11) Reuse Notes
- Primary target: SOAtest.
- API-first rule:
  - Author payload directly from endpoint schema (`tstsRequest`), not from existing workspace templates.
- If requirements traceability or tagging is requested, run Skill 009 on the root test suite immediately after creation.
