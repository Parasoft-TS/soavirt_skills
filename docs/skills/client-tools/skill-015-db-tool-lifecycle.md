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
Pure rename-only requests on existing DB Tools should route to `docs/skills/structure/skill-068-rename-object.md`, and pure copy/move/delete requests may route to the generic tool lifecycle leaves when no broader DB Tool configuration change is in scope.

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
  - connection values:
    - driver class
    - URL (`jdbc:{db}:{connectionstring}`) (for example `jdbc:hsqldb:hsql://localhost:9021/parabank`)
    - username
- Optional:
  - DB Tool name(s)
  - password (blank is allowed)
  - SQL query text (`toolSettings.sqlQuery.value.fixed`) (required for configured create; deferred in scaffold mode)
  - runtime/xml output options
  - copy/move target parent and optional renamed destination
### 4.0) Create Mode Selection (Required)
- Two create modes are supported:
  - `configured` (default): create a ready-to-run DB Tool with required connection and query inputs.
  - `scaffold-only`: create an empty/minimal DB Tool shell and defer operational fields.
- If the user does not explicitly request scaffold mode, proceed as `configured`.

### 4.1) Input Gate (Required)
- Never infer or silently apply connection/query values from prior examples.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context or active project record values that have been explicitly confirmed by the user for this branch.
- If create request is underspecified, pause creation and solicit missing values.
- In project-aware branches, active project facts, references, or environment-file values may be used as candidate DB configuration sources, but do not treat them as approved for non-scaffold create until the user or owning orchestration has explicitly confirmed them.
- Required before non-scaffold create:
  - `driver class`
  - `URL` (`jdbc:{db}:{connectionstring}`)
  - `username`
  - `SQL query text`
- `password` may remain empty/blank if user explicitly chooses no password.
- Do not switch to scaffold mode implicitly.

### 4.2) Conditional Required Matrix
- Create mode:
  - `configured` by default
  - `scaffold-only` only when user explicitly requests it
- Always required for create:
  - target parent suite id(s)
- Required for configured create:
  - driver class
  - URL (`jdbc:{db}:{connectionstring}`)
  - username
  - SQL query text
- Optional for all modes:
  - DB Tool name (derive deterministic name if absent)
  - password (blank allowed)

### 4.3) Missing-Field Solicitation Contract
When required fields are missing for the selected create mode, ask for all unresolved values in one prompt:
- Create mode (`configured` or `scaffold-only`; default `configured`):
- Driver class:
- URL (`jdbc:{db}:{connectionstring}`) (for example: `jdbc:hsqldb:hsql://localhost:9021/parabank`):
- Username:
- Password (optional, can be blank):
- SQL Query:
Do not call create endpoints until required values for the selected mode are provided/confirmed.

### 4.4) Scaffold-Only Create Mode
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
   - enforce Input Gate and Solicitation Contract before create.
   - branch by create mode:
     - configured (default):
       - require all configured-mode fields per Section 4.2 before create.
     - scaffold-only (explicit user request only):
       - allow minimal shell creation with deferred connection/query fields.
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

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in:
  - `docs/workflow/agent-workflow.md` (Decision Rule).
- Build DB Tool payloads from user input + REST API schema + this card's field model.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Rules:
- Replace all `{{...}}` placeholders from user-provided values and confirmed runtime intent.
- Do not reuse sample ids/names/driver/url/query literals unless explicitly requested.
- For updates, use `PUT /v6/tools/dbTools?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049.

Configured create shape:
```json
{
  "parent": {
    "id": "{{PARENT_SUITE_ID}}"
  },
  "name": "{{DB_TOOL_NAME}}",
  "toolSettings": {
    "connection": {
      "local": {
        "driver": "{{JDBC_DRIVER_CLASS}}",
        "url": "{{JDBC_URL}}",
        "username": "{{DB_USERNAME}}",
        "password": "{{DB_PASSWORD_OR_EMPTY}}"
      }
    },
    "sqlQuery": {
      "value": {
        "type": "fixed",
        "fixed": "{{SQL_QUERY_TEXT}}"
      }
    }
  }
}
```

Update shape (read-merge-write envelope):
```json
{
  "name": "{{DB_TOOL_NAME}}",
  "toolSettings": {
    "connection": {
      "local": {
        "driver": "{{JDBC_DRIVER_CLASS}}",
        "url": "{{JDBC_URL}}",
        "username": "{{DB_USERNAME}}",
        "password": "{{DB_PASSWORD_OR_EMPTY}}"
      }
    },
    "sqlQuery": {
      "value": {
        "type": "fixed",
        "fixed": "{{SQL_QUERY_TEXT}}"
      }
    }
  }
}
```

## 7) Validation
- Expected success pattern:
  - create/update/copy/move/read/delete calls return `200` on valid inputs.
- Post-condition checks:
  - tool id path reflects expected parent placement
  - readback shows configured local connection values
  - if create mode was scaffold-only, result is explicitly reported as incomplete with remaining fields listed.
  - temporary copy/move artifacts are absent after cleanup

## 8) Failure Modes
- `400` create/update for invalid payload shape or missing required parent/id.
- `404` when source/target ids do not exist.
- Ordering drift if `PUT /v6/suites/children` payload omits required siblings.
- Config misunderstanding risk:
  - API readback returns masked password representation even when empty input was used.
- Silent-default risk:
  - using example connection/query values when user did not provide them can create incorrect/unsafe configuration.
- Intent-mismatch risk:
  - creating scaffold when user intended configured (or vice versa) causes avoidable rework.

## 9) Safety / Rollback
- Write skill (mutates `.tst` structure).
- Rollback options:
  - delete newly created tools
  - move copied tools back to original parent
  - restore previous child order via saved `children` snapshot

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
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
