# Skill Card 003: Server-Side File Copy and Rename

## 1) Skill Name
Server-Side File Copy and Rename (`/v6/files/copy`)

## 2) Objective
- Create a server-side copy of an existing file and assign a new name without downloading/editing/re-uploading file content.

## 3) Scope
- In scope:
  - Copy a file to a target parent folder.
  - Set a new file name during copy.
  - Verify copied file exists.
- Out of scope:
  - Editing YAML content.
  - Moving files across environments.

## 4) Inputs
- Required:
  - `baseUrl` (for example `http://localhost:9080/soavirt/api/v6`)
  - `from.id` (source file id)
  - `to.parent.id` (destination folder id)
- Optional:
  - `to.name` (new filename for rename-on-copy)

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- Caller has permissions for read source and write destination.
- Source file id exists.

## 6) Procedure
1. (Optional) Check if target filename already exists under destination folder.
2. Call `POST /v6/files/copy` with body:
   - `from.id = <sourceId>`
   - `to.parent.id = <destinationFolderId>`
   - `to.name = <newFileName>`
3. Verify copied file via `GET /v6/descendants/files?id=<destinationFolderId>&type=<fileType>`.

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

## 8) Failure Modes
- `400`: malformed JSON body or invalid object structure.
- `404`: source `from.id` or destination `to.parent.id` does not exist.
- Name conflict in destination: copy fails if target name already exists.

## 9) Safety / Rollback
- Read-only by default? No (creates a new server file).
- Rollback plan for writes:
  - Delete mistakenly copied file via `DELETE /v6/files?id=<newId>`.

## 10) Reuse Notes
- Applies to SOAtest: yes (`.tst` files).
- Applies to Virtualize: yes (`.pva` files).
- Preferred over download/upload for copy/rename-only use cases because it avoids client-side encoding and payload round-trip risk.

## 11) Examples
- Example request body:
  - `{"from":{"id":"/TestAssets/Basic_Test_Construction_Learning.tst"},"to":{"name":"Basic_Test_Construction_Learning_New.tst","parent":{"id":"/TestAssets"}}}`
- Example endpoint call:
  - `POST http://localhost:9080/soavirt/api/v6/files/copy`
- Example result:
  - `id = /TestAssets/Basic_Test_Construction_Learning_New.tst`
