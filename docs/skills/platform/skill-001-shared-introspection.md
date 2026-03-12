# Skill Card 001: Shared File Introspection

## 1) Skill Name
Shared File Introspection (roots, folders, and typed descendants)

## 2) Objective
- Discover what files/assets are on the SOAVirt server using shared REST endpoints that apply to both SOAtest and Virtualize.

## 3) Scope
- In scope:
  - List workspace roots and immediate children.
  - Enumerate descendants under a folder.
  - Filter descendants by type (for example `tst`, `pva`, or `pvn`).
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
  - `type` (file type filter such as `tst`, `pva`, or `pvn`)

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- Caller has read access to workspace content.
## 5.1) Workspace Root Invariants (Practical)
- The server always exposes these workspace-root directories:
  - `/TestAssets`
  - `/VirtualAssets`
  - `/ProvisioningAssets`
- Treat these as the canonical base-path roots for server-hosted files.
- If the caller already provides a rooted server id/path under one of these directories, use it directly and do not relist roots first.
- If the caller provides only a filename or partial path, choose the most likely root first:
  - SOAtest `.tst` and related test assets -> `/TestAssets`
  - Virtualize `.pva` -> `/VirtualAssets`
  - Provisioning Assets `.pvn` and related Virtualize provisioning/support files -> `/ProvisioningAssets`
- Broaden to sibling roots only after the likely-root query returns no match.

## 6) Procedure
1. If root context is unresolved, call `GET /v6/children` with no `id` to confirm workspace roots and base-path reachability.
2. If the caller already has an exact rooted file/folder id under `/TestAssets`, `/VirtualAssets`, or `/ProvisioningAssets`, use that id directly.
3. If only a filename, file type, or partial path is known, choose the likely root first using Section 5.1.
4. Call `GET /v6/children?id=<id>` for immediate children when the target is likely a direct child of the chosen root.
5. Call `GET /v6/descendants/files?id=<id>` to retrieve recursive descendants only when immediate-child lookup is insufficient.
6. Add `type=<value>` (for example `tst`, `pva`, or `pvn`) to narrow recursive results.
7. For `.tst` structure reads, prefer:
  - `GET /v6/children?id=<tst-id>` for direct root-suite discovery,
  - `GET /v6/descendants/assets?id=<tst-id>` for broader asset context.
  - Do not assume class-specific file GET endpoints are available for structure reads.

### 6.1) Endpoint Selection Rule (Required)
- Use `GET /v6/descendants/files` for filesystem discovery only:
  - folders/files under workspace roots,
  - `.tst`/`.pva`/`.pvn` file resolution by path/name/type.
- Prefer one of the canonical roots (`/TestAssets`, `/VirtualAssets`, `/ProvisioningAssets`) as the recursive discovery base instead of broad or synthetic root ids.
- Use `GET /v6/children?id=<asset-id>` and `GET /v6/descendants/assets?id=<asset-id>` for asset-graph discovery only:
  - suites, tests, tools, datasources, environments, and other in-file objects.
- Do not use `GET /v6/descendants/assets` with directory ids (for example `/TestAssets`); resolve file ids first.
- Parse the collection field returned by the specific endpoint; do not assume every discovery response uses the `children` array.
### 6.2) Common Response Type Literals (Practical Reference)
- File/root discovery commonly returns API `type` values such as:
  - `directory`
  - `tst`
  - `pva`
  - `pvn`
- Asset-graph discovery commonly returns API `type` values such as:
  - `testSuite`
  - `restClient`
  - `excelDataSource`
  - `environment`
- These are API literals, not UI labels.
- When filtering or matching objects, use the observed `type` values returned by the API instead of guessing from display names such as "suite" or "REST Client".

### 6.3) Post-Mutation Nested Target Re-Resolution Rule (Shared)
- After a container-level mutation such as file copy or suite generation, treat nested object ids inside the new container as unstable unless the runtime/API explicitly proves id preservation.
- Do not assume that a nested source id continues to identify the copied/generated target object.
- Do not rely on one broad descendants query alone to guess the intended nested target when multiple similar objects may exist.
- Re-resolve nested targets from the new container root using this narrowing order:
  1. re-establish the copied/generated container root id exactly
  2. walk the stable structural path context captured before mutation (for example suite -> scenario -> test)
  3. narrow by expected object `type`
  4. narrow by stable local identity such as preserved name/label
  5. use preserved source-tree order only as a final tie-breaker when multiple otherwise-equivalent candidates remain
- If the local subtree cannot be re-established exactly or multiple candidates still remain after narrowing, fail closed and surface the ambiguity instead of guessing a nested target id.

## 7) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` invalid parameter
  - `401` unauthorized
  - `403` forbidden
  - `404` item not found
- Expected response shape:
  - `GET /v6/children`: JSON object with `children` array.
  - Descendants endpoints: JSON object with the endpoint-specific returned collection.
  - Returned objects include fields such as `id`, `name`, `type`, and `url`.
- Post-condition checks:
  - Root listing contains expected top folders (`TestAssets`, `VirtualAssets`, `ProvisioningAssets`).
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
- Provisioning usage: list `.pvn` under `/ProvisioningAssets` via `type=pvn`.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Shared components involved (e.g., JSON Data Bank): not required for this skill.

## 11) Examples
- Example request(s):
  - `GET http://localhost:9080/soavirt/api/v6/children`
  - `GET http://localhost:9080/soavirt/api/v6/children?id=%2FTestAssets`
  - `GET http://localhost:9080/soavirt/api/v6/descendants/files?id=%2FTestAssets&type=tst`
  - `GET http://localhost:9080/soavirt/api/v6/descendants/files?id=%2FVirtualAssets&type=pva`
  - `GET http://localhost:9080/soavirt/api/v6/descendants/files?id=%2FProvisioningAssets&type=pvn`
- Example response snippet(s):
  - `{"children":[{"id":"/TestAssets","name":"TestAssets","type":"directory"}]}`
