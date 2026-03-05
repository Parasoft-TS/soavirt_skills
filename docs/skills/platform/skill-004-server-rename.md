# Skill Card 004: Server-Side File Rename (In Place)

## 1) Skill Name
Server-Side File Rename (`PUT /v6/files`)

## 2) Objective
- Rename an existing file on the server without changing its content.

## 3) Scope
- In scope:
  - Rename a file by id using `name` in request body.
  - Verify old id is gone and new id exists.
- Out of scope:
  - Copying files.
  - Content edits.

## 4) Inputs
- Required:
  - `baseUrl`
  - `id` (current file id)
  - `name` (new filename)

## 5) Preconditions
- API reachable and authenticated.
- File id exists.
- Caller has write permission.

## 6) Procedure
1. Call `PUT /v6/files?id=<id>` with JSON body `{ "name": "<newName>" }`.
2. List descendants in parent folder to verify rename.

## 7) Validation
- Expected status: `200`.
- Verify old id not present and new id present in descendants listing.

## 8) Failure Modes
- `400` invalid request body/name.
- `404` file id not found.
- `403` permission/license restrictions.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback: call rename again to restore previous name.

## 10) Reuse Notes
- Applies to SOAtest and Virtualize file-level rename operations.

## 11) Example
- Request: `PUT /v6/files?id=%2FTestAssets%2FBasic_Test_Construction_Learning_New.tst`
- Body: `{ "name": "Basic_Test_Construction_Learning_Renamed.tst" }`
