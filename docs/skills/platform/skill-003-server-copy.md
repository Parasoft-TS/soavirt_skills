# Skill Card 003: Server-Side File Copy

## 1) Skill Name
Server-Side File Copy (with optional destination naming) (`POST /v6/files/copy`)

## 2) Objective
- Create a server-side copy of an existing file without downloading/editing/re-uploading file content.
- Optionally assign the destination copy a new file name during the copy operation.
- Use this card when a workflow needs a new artifact or a server-side failsafe copy before riskier local-edit work.

## 3) Scope
- In scope:
  - Copy a file to a target parent folder.
  - Same-folder or cross-folder copies when the destination parent is writable.
  - Set a destination file name during copy.
  - Verify the copied file exists and the source file remains unchanged.
- Out of scope:
  - In-place rename without creating a copy (use Skill 004).
  - Deleting copied artifacts (use Skill 005).
  - Editing YAML content.

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md` for collision checks and post-copy verification.

## 4) Inputs
- Required:
  - `baseUrl` (for example `http://localhost:9080/soavirt/api/v6`)
  - `from.id` (source file id)
  - `to.parent.id` (destination folder id)
- Optional:
  - `to.name` (destination filename for copy-on-create)

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- Caller has permissions for read source and write destination.
- Source file id exists.
- Exact destination parent id is resolved before mutation.

## 6) Procedure
1. (Optional) Check if target filename already exists under destination folder.
2. Call `POST /v6/files/copy` with body:
   - `from.id = <sourceId>`
   - `to.parent.id = <destinationFolderId>`
   - `to.name = <newFileName>`
3. Capture the returned copied-file metadata (`id`, `name`, `url`).
4. Verify copied file via destination-folder listing or exact-path readback.
5. Record both source id and copied id for downstream verification/cleanup.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid body/parameters
  - `401` unauthorized
  - `403` forbidden
  - `404` source or destination not found
- Expected response shape:
  - JSON with `id`, `name`, and `url` for copied file.
- Post-condition checks:
  - New file id exists at destination.
  - New filename matches expected `to.name`.
  - Source file remains present and unchanged.

## 8) Failure Modes
- `400`: malformed JSON body or invalid object structure.
- `404`: source `from.id` or destination `to.parent.id` does not exist.
- Name conflict in destination: copy fails if target name already exists.
- Ambiguous/truncated write output: reconcile via destination listing/readback before retry.

## 9) Safety / Rollback
- Read-only by default? No (creates a new server file).
- Rollback plan for writes:
  - Delete mistakenly copied file via Skill 005 (`DELETE /v6/files?id=<newId>`).

## 10) Reuse Notes
- SOAtest usage: copy `.tst` files.
- Virtualize usage: copy `.pva` files.
- Provisioning usage: copy `.pvn` files.
- Preferred over download/upload for copy-only or rollback-copy use cases because it avoids client-side encoding and payload round-trip risk.
- Use Skill 004 for in-place rename when no new artifact should be created.
- Use `docs/skills/backlog.md` for current validation and coverage status.

## 11) Examples
- Example request body:
  - `{"from":{"id":"/TestAssets/Basic_Test_Construction_Learning.tst"},"to":{"name":"Basic_Test_Construction_Learning_New.tst","parent":{"id":"/TestAssets"}}}`
- Example endpoint call:
  - `POST http://localhost:9080/soavirt/api/v6/files/copy`
- Example result:
  - `id = /TestAssets/Basic_Test_Construction_Learning_New.tst`
