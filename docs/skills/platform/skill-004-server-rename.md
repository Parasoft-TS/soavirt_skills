# Skill Card 004: Server-Side File Rename (In Place)

## 1) Skill Name
Server-Side File Rename (`PUT /v6/files`)

## 2) Objective
- Rename an existing file on the server without changing its content or creating a second artifact.

## 3) Scope
- In scope:
  - Rename a file by id using `name` in request body.
  - Verify old id is gone and new id exists.
- Out of scope:
  - Creating a new file copy (use Skill 003).
  - Content edits.
  - Delete/cleanup workflows (use Skill 005).

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md` for rename verification.

## 4) Inputs
- Required:
  - `baseUrl`
  - `id` (current file id)
  - `name` (new filename)

## 5) Preconditions
- API reachable and authenticated.
- File id exists.
- Caller has write permission.
- Exact target file id is resolved before mutation.

## 6) Procedure
1. Call `PUT /v6/files?id=<id>` with JSON body `{ "name": "<newName>" }`.
2. Re-list the parent folder and verify old id is absent and renamed id is present.
3. Record the prior name so rollback can rename back if needed.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid request body/name
  - `401` unauthorized
  - `403` forbidden
  - `404` file id not found
- Verify old id not present and new id present in parent-folder listing.

## 8) Failure Modes
- `400` invalid request body/name.
- `404` file id not found.
- `403` permission/license restrictions.
- Name conflict or invalid destination name prevents in-place rename.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback: call the same rename path again to restore the previous name.

## 10) Reuse Notes
- Applies to SOAtest and Virtualize file-level rename operations.
- Use Skill 003 instead when the workflow needs a second file or a rollback-safe copy.

## 11) Example
- Request: `PUT /v6/files?id=%2FTestAssets%2FBasic_Test_Construction_Learning_New.tst`
- Body: `{ "name": "Basic_Test_Construction_Learning_Renamed.tst" }`
