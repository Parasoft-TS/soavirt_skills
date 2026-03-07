# Skill Card 002: Shared File Transfer (Download/Upload)

## 1) Skill Name
Shared File Transfer (download for local edit, upload for server update)

## 2) Objective
- Transfer asset files between SOAVirt server and local workspace using shared REST endpoints so YAML edits can be made safely when needed.

## 3) Scope
- In scope:
  - Download file bytes/content by file id.
  - Upload local file bytes/content back to target file id.
  - Verify round-trip success using read-back checks.
- Out of scope:
  - Semantic editing of YAML internals.
  - Product-specific behavior beyond generic transfer.

## 3.1) Dependencies
- Required for write flows:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md` for pre/post transfer verification.

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
- Caller has read/write permission for target file id.
- For upload, local file exists and is readable.

## 6) Procedure
1. Download current server file with `GET /v6/files/download?id=<id>` and save locally.
2. (Optional) Create a backup copy of the downloaded file before editing.
3. Apply minimal local edits to the YAML.
4. Upload edited file using `POST /v6/files/upload?id=<id>&replace=true` as multipart form-data (`file`).
5. Re-download same id and confirm expected changes are present.

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
  - Re-downloaded content reflects intended edits only.
  - File remains parseable YAML.
  - No unexpected asset/folder relocation.

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
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Shared components involved (e.g., JSON Data Bank): transfer layer is shared regardless of tool type.

## 11) Examples
- Example request(s):
  - `GET http://localhost:9080/soavirt/api/v6/files/download?id=%2FTestAssets%2FBasic_Test_Construction_Learning.tst`
  - `POST http://localhost:9080/soavirt/api/v6/files/upload?id=%2FTestAssets%2FBasic_Test_Construction_Learning.tst&replace=true`
- Example command pattern (PowerShell):
  - `Invoke-WebRequest -UseBasicParsing -Uri "<downloadUrl>" -OutFile "<localPath>"`
  - `curl.exe -sS -o upload_response.json -w "%{http_code}" -X POST -F "file=@<localPath>" "<uploadUrl>"`
  - `([System.IO.File]::ReadAllBytes("<localPath>")[0..3] | ForEach-Object { $_.ToString('X2') }) -join ' '`
- Example response snippet(s):
  - Upload response contains file metadata for updated target.
