# Skill Card 005: Server-Side File Delete

## 1) Skill Name
Server-Side File Delete (`DELETE /v6/files`)

## 2) Objective
- Remove a file from the server by file id and verify the artifact is no longer present.

## 3) Scope
- In scope:
  - Delete a file id.
  - Verify deletion via parent-folder or exact-target listing.
- Out of scope:
  - Deleting non-empty folders recursively unless explicitly set.
  - Copy or rename operations.

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
- Target file/folder id is resolved before mutation.

## 6) Procedure
1. Call `DELETE /v6/files?id=<id>`.
2. Re-list the parent folder or exact target path and confirm file id absent.
3. Record the deleted id and any known recovery source if rollback may be needed.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid query parameters
  - `401` unauthorized
  - `403` forbidden
  - `404` id not found
- Verify deleted id no longer appears in verification listing.

## 8) Failure Modes
- `404` id not found.
- `403` permission/license restrictions.
- `400` invalid query parameters.
- Delete succeeds ambiguously/truncated: reconcile via readback before retry.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback: recover by restoring from a backup, source control, or a preserved server-side fallback copy when one exists.

## 10) Reuse Notes
- Applies to SOAtest and Virtualize file-level cleanup workflows.
- Common downstream use: cleanup of rollback/failsafe copies created by Skill 003 or Skill 006 workflows.

## 11) Example
- Request: `DELETE /v6/files?id=%2FTestAssets%2FBasic_Test_Construction_Learning_Renamed.tst`
