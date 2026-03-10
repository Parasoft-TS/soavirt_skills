# Skill 008: Data Source Type-Targeted Move

## 1) Skill Name
Data source type-targeted move (suite-to-suite)

## 2) Objective
Move exactly one selected data source implementation type (for example `TableDataSource`, `FileDataSource`, `WritableDataSource`) from one suite to another without moving sibling data source types.

## 3) Scope
- In scope:
  - SOAtest `.tst` suite-to-suite data source moves.
  - Type-targeted moves for `TableDataSource`, `FileDataSource`, `WritableDataSource`, `CSVDataSource`, `DbDataSource`, `ExcelDataSource`.
  - REST-first with YAML fallback when ids are implementation-ambiguous.
- Out of scope:
  - Cross-file moves between different `.tst` assets in one step.
  - Semantic edits to datasource internals beyond relocation.

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
- Required for YAML fallback branch:
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`

## 4) Inputs
- Required:
  - source file id (for example `/TestAssets/... .tst`)
  - source suite id
  - destination suite id or destination suite name + parent id
  - target implementation type (for example `TableDataSource`)
- Optional:
  - target datasource name in destination suite

## 5) Preconditions
- API base URL reachable.
- Auth requirements satisfied.
- File is restorable (snapshot available) before write operations.

## 6) Procedure
1. Resolve/create destination suite using `POST /v6/suites/testSuites` when needed.
2. Discover source suite children with `GET /v6/children?id=<source-suite-id>`.
3. Attempt REST move via `POST /v6/datasources/move` using candidate datasource id.
4. Download YAML and verify whether the target implementation type moved.
5. If REST move selected a different implementation (ambiguous shared id), invoke Skill 006 for the YAML fallback branch:
   - establish rollback-preserving fallback copy/local rollback sources
   - perform only the surgical YAML edit owned by this card
   - upload/read back/restore/cleanup through the Skill 006 workflow
6. For the YAML fallback edit itself, move only the datasource block where `impl.$type == <target-type>`.
7. Re-download and verify implementation type distribution by suite.

## 7) Validation
- Expected HTTP status codes:
  - `200` for suite creation/move/upload success
  - `400/404` possible when ids/parents are invalid
- Expected response shape:
  - move/create responses include `id`, `name`, `url`
- Post-condition checks:
  - source suite no longer contains target implementation type
  - destination suite contains target implementation type
  - non-target sibling datasource types remain in source suite

## 8) Failure Modes
- Shared datasource id maps to multiple implementations and REST move moves a different one first.
- `children` view shows duplicate generic `dataSource` entries without type specificity.
- Upload fails if `id` is not provided as query parameter on this server.

## 9) Safety / Rollback
- Read-only by default? No (this is a write skill).
- Rollback plan for writes:
  - for REST move branch, verify target-type distribution immediately and stop if the wrong implementation moved
  - for YAML fallback branch, use the rollback-preserving workflow in Skill 006

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize object models may differ and are outside this card's default scope.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Shared components involved: `GET/POST /v6/suites/testSuites`, `POST /v6/datasources/move`, `GET /v6/files/download`, `POST /v6/files/upload`.
- The YAML fallback branch should route through `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md` rather than re-documenting copy/upload/restore choreography locally.
- Foundational dependency:
  - `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`
  - Use it before mutation to confirm available datasource families/columns and to improve move-impact analysis for downstream parameterized tests.

## 11) Examples
- Example intent:
  - Move only `WritableDataSource` from `Source Suite` to `Target Suite`.
- Example verification:
  - Source types before: `CSV, Db, Excel, File, Writable`
  - Source types after: `CSV, Db, Excel, File`
  - Target types after: includes `Writable`
