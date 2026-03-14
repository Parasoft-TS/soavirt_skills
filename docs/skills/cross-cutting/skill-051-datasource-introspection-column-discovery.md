# Skill 051: Data Source Introspection and Column Discovery

## 1) Skill Name
Data source introspection + column discovery (read-only foundation)

## 2) Objective
Provide a reusable, deterministic read-only workflow to answer:
- what data sources are available in a target `.tst`/suite/tool context,
- what type each data source is,
- which columns are available for parameterization,
- which row samples are available for confidence checks.

This skill is a shared foundation for:
- configuration analysis/dataflow tracing,
- datasource-aware parameterization workflows,
- datasource move/refactor preflight and impact checks.

## 3) Scope
- In scope:
  - discover datasource nodes under a target `.tst` or suite.
  - classify datasource type and resolve class-specific datasource detail endpoint.
  - resolve column names when supported by endpoint family.
  - optionally resolve sample row data (`limit`/`offset`) for diagnostics.
  - return normalized output with confidence + fallback reason.
- Out of scope:
  - creating/updating/moving/deleting data sources.
  - changing test parameterization.
  - semantic business interpretation of datasource contents.

## 4) Inputs
- Required:
  - target `.tst` file id (for example `/TestAssets/... .tst`)
- Optional:
  - focus suite id (to scope visible data sources)
  - include sample row data (`true|false`)
  - sample row limit (default: small bounded value)

## 5) Preconditions
- API reachable and authenticated.
- Target `.tst` and optional suite id exist.
- Read-only mode (no write endpoints).

## 6) Procedure
1. Discover candidate data source nodes in target scope:
  - `GET /v6/descendants/assets?id=<tst-id>` (or suite-subtree scope by filtering ids).
  - collect nodes where `type=dataSource` (and specialized datasource node types when present).
2. For each datasource candidate, resolve detailed configuration:
  - attempt class-specific GET endpoint based on datasource family:
    - CSV: `GET /v6/datasources/csv?id=...`
    - Database: `GET /v6/datasources/database?id=...`
    - Excel: `GET /v6/datasources/excel?id=...`
    - Database Correlation: `GET /v6/datasources/databaseCorrelation?id=...`
    - Data Group: `GET /v6/datasources/dataGroup?id=...`
    - Repository: `GET /v6/datasources/repository?id=...`
3. Resolve columns where supported:
  - CSV: `GET /v6/datasources/csv/columns?id=...`
  - Database: `GET /v6/datasources/database/columns?id=...`
  - Excel: `GET /v6/datasources/excel/columns?id=...`
  - Database Correlation: `GET /v6/datasources/databaseCorrelation/columns?id=...`
4. Optionally resolve sample data rows where supported:
  - CSV: `GET /v6/datasources/csv/data?id=...&limit=...&offset=...`
  - Database: `GET /v6/datasources/database/data?id=...&limit=...&offset=...`
  - Excel: `GET /v6/datasources/excel/data?id=...&limit=...&offset=...`
  - Database Correlation: `GET /v6/datasources/databaseCorrelation/data?id=...`
5. Emit normalized per-datasource records:
  - `id`, `name`, `family`, `parent`, `columns[]`, `sampleRows[]`, `confidence`, `fallbackReason`.
6. If family endpoint support is missing or unresolved:
  - keep datasource in output with `columns=Unknown`, `confidence=partial`, and explicit reason.

## 6.1) Capability Matrix Rule (Required)
- Always treat column/data endpoint support as family-specific, not universal.
- Do not infer columns from display names when column endpoints are unavailable.
- For unsupported families, report explicitly as unresolved rather than guessed.

## 7) Validation
- Expected HTTP status codes:
  - `200` on successful introspection calls.
  - `400/404` possible for invalid id or family mismatch.
- Post-condition checks:
  - each discovered datasource has a normalized output record.
  - each resolved column list cites the endpoint family used.
  - unresolved records include explicit fallback reason.

## 8) Failure Modes
- Ambiguous datasource ids when REST hierarchy uses generic `dataSource` nodes.
- Family mismatch (`id` valid but queried on wrong family endpoint) returns `400/404`.
- Partial coverage when some datasource families have no column/data endpoints.
- Database correlation data rows may require valid sample correlation keys.

## 9) Safety / Rollback
- Read-only by default? Yes.
- No rollback required (no mutation endpoints).

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
- Shared endpoint set:
  - `GET /v6/descendants/assets`
  - `GET /v6/datasources/*`
  - `GET /v6/datasources/*/columns`
  - `GET /v6/datasources/*/data`
- Intended downstream consumers:
  - dataflow/configuration analysis skill,
  - datasource-aware parameterization skill,
  - structural datasource move/refactor workflows (preflight and impact checks).
