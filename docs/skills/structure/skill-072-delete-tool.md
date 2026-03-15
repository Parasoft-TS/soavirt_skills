# Skill 072: Delete Tool
## 1) Skill Name
Delete one existing tool through the generic `DELETE /v6/tools` API.
## 2) Objective
Provide one atomic owner for pure tool deletion so agents use the supported generic tool delete path and deterministic readback reconciliation.
## 3) Scope
- In scope:
  - delete one existing tool via `DELETE /v6/tools?id=...`
  - post-delete absence verification
- Out of scope:
  - suite, datasource, environment, responder, or file deletion
  - local YAML deletion of tool blocks as a substitute for the supported API route
## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 4) Inputs
- Required:
  - tool id
- Optional:
  - expected class-specific endpoint for absence verification
  - expected parent id for child-list confirmation
## 5) Preconditions
- The tool id resolves before delete.
- Capability preflight confirms `DELETE /v6/tools`.
## 6) Procedure
1. Read the target tool before delete through either its known class-specific endpoint or generic `GET /v6/tools?id=<tool-id>`.
2. Capture enough pre-delete identity to verify absence afterward.
3. Execute `DELETE /v6/tools?id=<tool-id>`.
4. Verify deletion with deterministic readback reconciliation:
   - first retry the same read route and expect absence or `404`
   - if the owning workflow also has parent context, confirm the deleted tool no longer appears under the parent's children/relationships
5. If delete output is ambiguous, do not retry immediately; reconcile state first and retry only if readback proves the tool still exists.
## 7) Validation
- Delete uses the generic tool delete route.
- Post-delete verification proves absence rather than inferring success from command output alone.
## 8) Failure Modes
- Blindly retrying delete after ambiguous output without first reconciling state.
- Accepting partial/truncated delete output as success without readback.
- Deleting by local YAML block removal while a supported API path exists.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - there is no generic undelete path; capture enough pre-delete identity for reporting and stop if delete confirmation remains ambiguous
## 10) Reuse Notes
- Use this card for pure tool-delete intent.
- Broader lifecycle cards may still invoke the same generic delete route when delete happens as part of a larger approved workflow.
