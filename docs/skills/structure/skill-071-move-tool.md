# Skill 071: Move Tool
## 1) Skill Name
Move one existing tool to a different parent through the generic `/v6/tools/move` API.
## 2) Objective
Provide one atomic owner for pure tool move/reparent requests so agents do not fall back to local YAML edits for ordinary tool relocation.
## 3) Scope
- In scope:
  - move one existing tool via `POST /v6/tools/move`
  - optional destination name override via `to.name`
  - post-move verification of final parent placement and final name
- Out of scope:
  - suite, datasource, environment, or file moves
  - operation-specific YAML surgery
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
- Capability preflight confirms `POST /v6/tools/move`.
## 6) Procedure
1. Resolve the source tool through its class-specific endpoint when known, or generic `GET /v6/tools?id=<from.id>` when only a tool id is available.
2. Preserve the pre-move readback as verification context, including old parent relationship if exposed.
3. Submit `POST /v6/tools/move` with:
   - `from.id`
   - `to.parent.id`
   - optional `to.name`
4. Capture the move response `id`, `name`, and `url`.
5. Read back the moved tool through the returned class-specific `url` when available; otherwise derive the class-specific endpoint from the source class.
6. Confirm postconditions:
   - the moved tool is readable after the move
   - the final parent relationship matches `to.parent.id` when exposed by readback
   - the final name matches `to.name` when provided, otherwise remains stable
7. If the returned/readback identity is ambiguous, fail closed instead of inferring the move from local file structure.
## 7) Validation
- Move uses the generic `/v6/tools/move` route.
- Verification reads the moved object after the move.
- Parent placement is confirmed when the owning readback exposes relationships.
## 8) Failure Modes
- Assuming the source parent changed without rereading the moved tool.
- Accepting a name override without verifying the returned/readback name.
- Performing move by local YAML block relocation when the generic API path exists.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - preserve the pre-move parent context
  - if verification fails after a successful move call, restore by invoking the same move owner back to the original parent rather than editing YAML
## 10) Reuse Notes
- Use this card for pure tool reparenting intent.
- Keep `docs/skills/structure/skill-008-datasource-type-targeted-move.md` for its narrower datasource ambiguity case; do not conflate that YAML-authorized exception with ordinary tool moves.
