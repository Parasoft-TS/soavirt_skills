# Skill Card 001: Shared File Introspection

## 1) Skill Name
Shared File Introspection (roots, folders, and typed descendants)

## 2) Objective
- Discover what files/assets are on the SOAVirt server using shared REST endpoints that apply to both SOAtest and Virtualize.

## 3) Scope
- In scope:
  - List workspace roots and immediate children.
  - Enumerate descendants under a folder.
  - Filter descendants by type (for example `tst` or `pva`).
  - Resolve `.tst` structural children (for example root suite) using compatibility-safe paths.
- Out of scope:
  - Downloading file content.
  - Uploading changes.
  - Editing asset internals.

## 3.1) Dependencies
- Required:
  - none
- Additive:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` when this skill is used as a runtime capability-preflight or target-resolution foundation.

## 4) Inputs
- Required:
  - `baseUrl` (for example `http://localhost:9080/soavirt/api/v6`)
- Optional:
  - `id` (folder or file id such as `/TestAssets`)
  - `type` (file type filter such as `tst` or `pva`)

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- Caller has read access to workspace content.

## 6) Procedure
1. Call `GET /v6/children` with no `id` to list workspace roots.
2. Choose a folder id and call `GET /v6/children?id=<id>` for immediate children if needed.
3. Call `GET /v6/descendants/files?id=<id>` to retrieve recursive descendants.
4. Add `type=<value>` (for example `tst` or `pva`) to narrow results.
5. For `.tst` structure reads, prefer:
  - `GET /v6/children?id=<tst-id>` for direct root-suite discovery,
  - `GET /v6/descendants/assets?id=<tst-id>` for broader asset context.
  - Do not assume class-specific file GET endpoints are available for structure reads.

### 6.1) Endpoint Selection Rule (Required)
- Use `GET /v6/descendants/files` for filesystem discovery only:
  - folders/files under workspace roots,
  - `.tst`/`.pva` file resolution by path/name/type.
- Use `GET /v6/children?id=<asset-id>` and `GET /v6/descendants/assets?id=<asset-id>` for asset-graph discovery only:
  - suites, tests, tools, datasources, environments, and other in-file objects.
- Do not use `GET /v6/descendants/assets` with directory ids (for example `/TestAssets`); resolve file ids first.
- Parse response payloads from the `children` array for both descendants endpoints.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid parameter
  - `401` unauthorized
  - `403` forbidden
  - `404` item not found
- Expected response shape:
  - JSON object with `children` array.
  - Child objects include fields such as `id`, `name`, `type`, and `url`.
- Post-condition checks:
  - Root listing contains expected top folders (for example `TestAssets`, `VirtualAssets`).
  - Type-filtered queries return only requested file type entries.

## 8) Failure Modes
- Empty result set: wrong `id`, wrong `type`, or no matching files.
- `400`: malformed query parameters.
- `401` / `403`: credentials or permission issue.
- `404`: resource id does not exist.

## 9) Safety / Rollback
- Read-only by default? Yes
- Rollback plan for writes: Not applicable (no writes in this skill).

## 10) Reuse Notes
- SOAtest usage: list `.tst` under `/TestAssets` via `type=tst`.
- Virtualize usage: list `.pva` under `/VirtualAssets` via `type=pva`.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Shared components involved (e.g., JSON Data Bank): not required for this skill.

## 11) Examples
- Example request(s):
  - `GET http://localhost:9080/soavirt/api/v6/children`
  - `GET http://localhost:9080/soavirt/api/v6/descendants/files?id=%2FTestAssets&type=tst`
  - `GET http://localhost:9080/soavirt/api/v6/descendants/files?id=%2FVirtualAssets&type=pva`
- Example response snippet(s):
  - `{"children":[{"id":"/TestAssets","name":"TestAssets","type":"directory"}]}`
