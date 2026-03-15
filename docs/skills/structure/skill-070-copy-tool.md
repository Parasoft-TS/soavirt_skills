# Skill 070: Copy Tool
## 1) Skill Name
Copy one existing tool through the generic `/v6/tools/copy` API.
## 2) Objective
Provide one atomic owner for pure tool copy requests so agents use the generic tool lifecycle API rather than ad hoc file edits or over-broad class-specific workflows.
## 3) Scope
- In scope:
  - copy one existing tool via `POST /v6/tools/copy`
  - optional destination name override via `to.name`
  - post-copy verification of identity, placement, and name
- Out of scope:
  - copy of suites, data sources, environments, or files
  - broader class-specific configuration edits to the copied tool
## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 4) Inputs
- Required:
  - `from.id`
  - `to.parent.id`
- Optional:
  - `to.name`
  - expected source tool class for verification routing
## 5) Preconditions
- The source tool id resolves.
- The destination parent id resolves.
- Capability preflight confirms `POST /v6/tools/copy`.
## 6) Procedure
1. Resolve the source tool through either its known class-specific endpoint or generic `GET /v6/tools?id=<from.id>` when the class-specific URL is not already known.
2. Capture the source identity needed for verification:
   - source id
   - source name
   - source class-specific URL if available
3. Submit `POST /v6/tools/copy` with:
   - `from.id`
   - `to.parent.id`
   - optional `to.name`
4. Capture the copy response `id`, `name`, and `url`.
5. Read back the new tool through the returned class-specific `url` when available; otherwise derive the class-specific endpoint from the source class.
6. Confirm postconditions:
   - the copied tool id is present and readable
   - the copied tool id is not the same as the source id
   - the copied tool name matches `to.name` when provided, otherwise the returned/readback name is stable
   - the copied tool now resolves under the intended parent relationship when that relationship is exposed by readback
## 7) Validation
- Copy uses the generic `/v6/tools/copy` route.
- Verification reads the new copied object rather than assuming the response is sufficient by itself.
- Parent placement is confirmed when readback exposes relationships.
## 8) Failure Modes
- Treating the copy response alone as sufficient without rereading the copied tool.
- Assuming the copied tool preserves or changes ids without checking the response.
- Performing copy by local YAML block duplication.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - if the copy lands incorrectly but is readable, remove the copied tool with the delete-tool owner rather than editing local YAML
## 10) Reuse Notes
- Use this card for pure copy intent on existing tools.
- If a class-specific lifecycle card is already active for broader create/update/configuration work, that card may still invoke the same generic copy endpoint as a substep.
