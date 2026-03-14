# Skill Card 002: Shared File Transfer (Download/Upload)

## 1) Skill Name
Shared File Transfer (download for local edit, upload for server update)

## 2) Objective
- Transfer asset files between SOAVirt server and local workspace using shared REST endpoints so local inspection or edits can be made safely when needed.

## 3) Scope
- In scope:
  - Download file bytes/content by file id.
  - Upload local file bytes/content back to target file id.
  - Verify round-trip success using read-back checks.
- Out of scope:
  - Semantic editing of YAML internals.
  - Product-specific behavior beyond generic transfer.

## 3.1) Dependencies
- Required when target file id is unresolved or only a filename/partial path is known:
  - `docs/skills/platform/skill-001-shared-introspection.md`
- Required for write flows:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 3.2) Canonical Root Awareness
- These server root directories are always present and act as the base-path roots for server files:
  - `/TestAssets`
  - `/VirtualAssets`
  - `/ProvisioningAssets`
- If `id` already starts with one of these rooted paths, use it directly and skip root rediscovery.
- If only a filename or partial path is known, resolve the exact rooted `id` first via Skill 001 using the likely-root-first rule:
  - SOAtest `.tst` and related test assets -> `/TestAssets`
  - Virtualize `.pva` -> `/VirtualAssets`
  - Provisioning Assets `.pvn` and related provisioning/support files -> `/ProvisioningAssets`

## 4) Inputs
- Required:
  - `baseUrl` (for example `http://localhost:9080/soavirt/api/v6`)
  - `id` (file id such as `/TestAssets/example.tst`)
  - `localPath` (local file path for download or upload)
- Optional:
  - `replace` (upload replacement behavior, default false)
  - `deploy` (upload deploy behavior, default true; mainly relevant for `.pva`)

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- Caller has read permission for the target file id.
- For upload/write-back flows, caller also has write permission for the target file id.
- For upload, local file exists and is readable.

## 6) Procedure
1. Resolve the exact rooted file `id` first.
2. Download current server file with `GET /v6/files/download?id=<id>` and save locally.
3. If the workflow is read-only, stop after download and perform any local parse/hash/content checks needed by the consuming workflow.
4. If the workflow includes edits, create a backup copy of the downloaded file before editing.
5. Apply minimal local edits to the file content.
6. Upload edited file using `POST /v6/files/upload?id=<id>&replace=true` as multipart form-data (`file`).
7. Re-download same id and confirm expected changes are present.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid parameter or payload
  - `401` unauthorized
  - `403` forbidden
  - `404` item not found
- Expected response shape:
  - Download: binary/octet-stream response body.
  - Upload: JSON response with file metadata.
- Post-condition checks:
  - Download-only branch: local content is readable/parseable for the consuming workflow.
  - Re-downloaded content reflects intended edits only.
  - File remains parseable in its expected format (for example YAML when a YAML-backed workflow is in scope).
  - No unexpected asset/folder/root relocation.

## 8) Failure Modes
- `400` on upload: malformed multipart payload or incompatible content.
- `403`: write not allowed or license restrictions.
- `404`: target id invalid.
- Upload succeeds but content unchanged: wrong target `id` or `replace` behavior not enabled.
- `StreamCorruptedException: invalid stream header: EFBBBF2D`: YAML was saved with UTF-8 BOM before upload.

## 8.1) Encoding Safety (Critical)
- Always write edited YAML as UTF-8 **without BOM**.
- Before upload, verify first bytes are `2D 2D 2D` (`---`), not `EF BB BF 2D`.
- Keep original downloaded bytes as rollback source.
- For PowerShell, use explicit no-BOM encoding when writing files:
  - `[System.IO.File]::WriteAllText($path, $text, (New-Object System.Text.UTF8Encoding($false)))`

## 9) Safety / Rollback
- Read-only by default? No (this skill includes writes).
- Rollback plan for writes:
  - Always keep a pre-edit backup copy.
  - If upload result is wrong, immediately re-upload backup file to same `id` with `replace=true`.
  - Re-download and verify rollback content.

## 10) Reuse Notes
- SOAtest usage: transfer `.tst` files under `/TestAssets`.
- Virtualize usage: transfer `.pva` files under `/VirtualAssets`.
- Provisioning usage: transfer `.pvn` files under `/ProvisioningAssets`.
- Shared components involved (e.g., JSON Data Bank): transfer layer is shared regardless of tool type.

## 11) Examples
- Example request(s):
  - `GET http://localhost:9080/soavirt/api/v6/files/download?id=%2FTestAssets%2FBasic_Test_Construction_Learning.tst`
  - `POST http://localhost:9080/soavirt/api/v6/files/upload?id=%2FTestAssets%2FBasic_Test_Construction_Learning.tst&replace=true`
  - `GET http://localhost:9080/soavirt/api/v6/files/download?id=%2FProvisioningAssets%2Fexample.pvn`
  - `POST http://localhost:9080/soavirt/api/v6/files/upload?id=%2FProvisioningAssets%2Fexample.pvn&replace=true`
- Example command pattern (PowerShell):
  - `Invoke-WebRequest -UseBasicParsing -Uri "<downloadUrl>" -OutFile "<localPath>"`
  - `curl.exe -sS -o upload_response.json -w "%{http_code}" -X POST -F "file=@<localPath>" "<uploadUrl>"`
  - `([System.IO.File]::ReadAllBytes("<localPath>")[0..3] | ForEach-Object { $_.ToString('X2') }) -join ' '`
- Example response snippet(s):
  - Upload response contains file metadata for updated target.
