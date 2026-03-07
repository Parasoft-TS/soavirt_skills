# Skill Card 005: Server-Side File Delete

## 1) Skill Name
Server-Side File Delete (`DELETE /v6/files`)

## 2) Objective
- Remove a file from the server by file id.

## 3) Scope
- In scope:
  - Delete a file id.
  - Verify deletion via descendants listing.
- Out of scope:
  - Deleting non-empty folders recursively unless explicitly set.

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md` for deletion verification.

## 4) Inputs
- Required:
  - `baseUrl`
  - `id` (file id to delete)
- Optional:
  - `recursive` (for folder delete scenarios)

## 5) Preconditions
- API reachable and authenticated.
- File id exists.
- Caller has delete permission.

## 6) Procedure
1. Call `DELETE /v6/files?id=<id>`.
2. Re-list descendants in parent folder and confirm file id absent.

## 7) Validation
- Expected status: `200`.
- Verify deleted id no longer appears in listing.

## 8) Failure Modes
- `404` id not found.
- `403` permission/license restrictions.
- `400` invalid query parameters.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback: recover by creating/copying/restoring from source control or backup.

## 10) Reuse Notes
- Applies to SOAtest and Virtualize file-level cleanup workflows.

## 11) Example
- Request: `DELETE /v6/files?id=%2FTestAssets%2FBasic_Test_Construction_Learning_Renamed.tst`
