# DB Tool Lifecycle Skill Card
## Update Policy
Updates to DB Tool objects must follow baseline gating, output chaining, and read-merge-write update rules. See Skill 017 for output chaining model and Skill 049 for mutation safety policy.

## Capability Preflight
Before performing any mutation or lifecycle operation, perform capability preflight checks as described in Skill 050.

## Cross-Cutting Dependencies
Depends on:
- `docs/skills/cross-cutting/skill-017-output-chaining-model.md` (output chaining)
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` (read-merge-write update)
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` (capability preflight)

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

### 4.1) Input Gate (Required)
- Never infer or silently apply connection/query values from prior examples.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context that has been explicitly confirmed by the user.
- If create request is underspecified, pause creation and solicit missing values.
- Required before non-scaffold create:
  - `driver`
  - `JDBC URL`
  - `username`
  - `SQL query text`
- `password` may remain empty if user explicitly chooses no password.

### 4.2) Missing-Field Solicitation Contract
When any required create field is missing, ask for all unresolved values in one prompt:
- Driver:
- JDBC URL:
- Username:
- Password (optional, can be blank):
- SQL query text:
Do not call create endpoints until required values are provided/confirmed.

### 4.3) Scaffold-Only Create Mode
- Supported only when user explicitly requests a blank/scaffold DB Tool.
- In scaffold mode:
  - create with empty/minimal connection/query placeholders,
  - label tool name to indicate scaffold status if requested,
  - immediately report that configuration is incomplete and list fields to fill.
- Do not enter scaffold mode implicitly.

## 5) Preconditions
- API reachable and authenticated.
- Parent suite ids exist.
- If deterministic ordering is required, full root-child ordering can be read first.

## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile E for tool mutation steps,
   - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Inspect current placement context:
   - `GET /v6/children?id=<suite-id>`
2. Create DB Tool(s):
   - enforce Input Gate and Solicitation Contract before create (unless explicit scaffold-only mode is requested).
   - `POST /v6/tools/dbTools` with:
     - `parent.id`
     - `name`
     - `toolSettings.connection.local.driver`
     - `toolSettings.connection.local.url`
     - `toolSettings.connection.local.username`
     - `toolSettings.sqlQuery.value.fixed` (required outside scaffold mode)
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
- Silent-default risk:
  - using example connection/query values when user did not provide them can create incorrect/unsafe configuration.

## 9) Safety / Rollback
- Write skill (mutates `.tst` structure).
- Rollback options:
  - delete newly created tools
  - move copied tools back to original parent
  - restore previous child order via saved `children` snapshot

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Related endpoints:
  - `GET/PUT/DELETE /v6/tools`
  - `POST /v6/tools/copy`
  - `POST /v6/tools/move`
  - `POST/PUT/GET /v6/tools/dbTools`
  - `GET/PUT /v6/suites/children`
- Cross-cutting dependency for chained downstream tools:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Chain assertors/validators to semantic DB outputs (for example `Results as XML`) rather than diagnostics-first traffic outputs.
## 11) Example Configuration Pattern
Example DB Tool connection fields (example only, not implicit defaults):
Example validated DB Tool connection fields (example only, not implicit defaults):
- `toolSettings.connection.local.driver = org.hsqldb.jdbcDriver`
- `toolSettings.connection.local.url = jdbc:hsqldb:hsql://localhost:9021/parabank`
- `toolSettings.connection.local.username = sa`
- SQL query field example:
  - `toolSettings.sqlQuery.value.fixed = select * from Account where ID=12345`
