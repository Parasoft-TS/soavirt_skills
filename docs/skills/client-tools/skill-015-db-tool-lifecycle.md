# DB Tool Lifecycle Skill Card
## Update Policy
Updates to DB Tool objects must follow baseline gating, output chaining, and read-merge-write update rules. See Skill 017 for output chaining model and Skill 049 for mutation safety policy.

## Capability Preflight
Before performing any mutation or lifecycle operation, perform capability preflight checks as described in Skill 050.

## Cross-Cutting Dependencies
Depends on: Skill 017 (output chaining), Skill 049 (read-merge-write update), Skill 050 (capability preflight)

# Skill 015: DB Tool Lifecycle (Create/Update/Copy/Move/Delete)

## 1) Skill Name
DB Tool create / update / copy / move / delete

## 2) Objective
Manage SOAtest DB Tools in `.tst` assets with deterministic placement and validated JDBC connection configuration.

## 3) Scope
- In scope:
  - create DB Tool via `POST /v6/tools/dbTools`
  - update DB Tool via `PUT /v6/tools/dbTools?id=...`
  - read DB Tool via `GET /v6/tools/dbTools?id=...`
  - copy/move DB Tool via `POST /v6/tools/copy` and `POST /v6/tools/move`
  - delete DB Tool via `DELETE /v6/tools?id=...`
  - positional placement under suites using `PUT /v6/suites/children?id=...`
- Out of scope:
  - SQL query authoring strategy and data-model design
  - database-side setup/remediation

## 4) Inputs
- Required:
  - target parent suite id(s)
  - DB Tool name(s)
  - connection values:
    - driver
    - JDBC URL
    - username
- Optional:
  - password
  - SQL query text (`toolSettings.sqlQuery.value.fixed`)
  - runtime/xml output options
  - copy/move target parent and optional renamed destination

## 5) Preconditions
- API reachable and authenticated.
- Parent suite ids exist.
- If deterministic ordering is required, full root-child ordering can be read first.

## 6) Procedure
1. Inspect current placement context:
   - `GET /v6/children?id=<suite-id>`
2. Create DB Tool(s):
   - `POST /v6/tools/dbTools` with:
     - `parent.id`
     - `name`
     - `toolSettings.connection.local.driver`
     - `toolSettings.connection.local.url`
     - `toolSettings.connection.local.username`
3. If exact order is required among suite children:
   - compute full ordered child id list
   - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id` sequence
4. Read back created tool(s):
   - `GET /v6/tools/dbTools?id=<tool-id>`
5. Update tool configuration/name as needed:
   - `PUT /v6/tools/dbTools?id=<tool-id>`
6. Copy and/or move tool when restructuring:
   - copy: `POST /v6/tools/copy` with `{ from.id, to.parent.id, to.name? }`
   - move: `POST /v6/tools/move` with `{ from.id, to.parent.id, to.name? }`
7. Delete temporary/obsolete tools:
   - `DELETE /v6/tools?id=<tool-id>`

## 7) Validation
- Expected success pattern:
  - create/update/copy/move/read/delete calls return `200` on valid inputs.
- Post-condition checks:
  - tool id path reflects expected parent placement
  - readback shows configured local connection values
  - temporary copy/move artifacts are absent after cleanup

## 8) Failure Modes
- `400` create/update for invalid payload shape or missing required parent/id.
- `404` when source/target ids do not exist.
- Ordering drift if `PUT /v6/suites/children` payload omits required siblings.
- Config misunderstanding risk:
  - API readback returns masked password representation even when empty input was used.

## 9) Safety / Rollback
- Write skill (mutates `.tst` structure).
- Rollback options:
  - delete newly created tools
  - move copied tools back to original parent
  - restore previous child order via saved `children` snapshot

## 10) Reuse Notes
- Applies to SOAtest: Yes (validated).
- Applies to Virtualize: not validated.
- Related endpoints:
  - `GET/PUT/DELETE /v6/tools`
  - `POST /v6/tools/copy`
  - `POST /v6/tools/move`
  - `POST/PUT/GET /v6/tools/dbTools`
  - `GET/PUT /v6/suites/children`
- Cross-cutting dependency for chained downstream tools:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Chain assertors/validators to semantic DB outputs (for example `Results as XML`) rather than diagnostics-first traffic outputs.

## 11) Validated Configuration Pattern
Validated DB Tool connection fields:
- `toolSettings.connection.local.driver = org.hsqldb.jdbcDriver`
- `toolSettings.connection.local.url = jdbc:hsqldb:hsql://localhost:9021/parabank`
- `toolSettings.connection.local.username = sa`
- validated SQL query field:
  - `toolSettings.sqlQuery.value.fixed = select * from Account where ID=12345`

## 12) Current Validation Status (2026-03-03)
- Created at root suite and placed between root REST client test and first child suite:
  - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/DB Tool - Root Between Test And Child`
- Created inside first child suite:
  - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/Child Test Suite/DB Tool - First Child Suite`
- Validated lifecycle operations:
  - update (`PUT /v6/tools/dbTools`)
  - copy (`POST /v6/tools/copy`)
  - move (`POST /v6/tools/move`)
  - delete (`DELETE /v6/tools`)
- Validated SQL query update/readback on both created tools:
  - `select * from Account where ID=12345`
- Evidence folder:
  - `work/runs/2026-03-03/tst-current/db-tool-skill-build/`
