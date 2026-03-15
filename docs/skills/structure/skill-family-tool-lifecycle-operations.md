# Skill Family: Generic Tool Lifecycle Operations
Purpose: route pure tool copy, move, and delete requests to the generic `/v6/tools*` lifecycle owners.
## Scope
This family covers tool-agnostic lifecycle operations that the generic tools API truly owns:
- copy tool
- move tool
- delete tool
It does not own rename, disabled-state mutation, or class-specific configuration updates.
## Foundational Skills
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- `docs/skills/structure/skill-070-copy-tool.md`
- `docs/skills/structure/skill-071-move-tool.md`
- `docs/skills/structure/skill-072-delete-tool.md`
## Selection Guide
- Need to duplicate one existing tool under another parent?
  - use `docs/skills/structure/skill-070-copy-tool.md`
- Need to reparent one existing tool to another parent?
  - use `docs/skills/structure/skill-071-move-tool.md`
- Need to remove one existing tool?
  - use `docs/skills/structure/skill-072-delete-tool.md`
- Need to rename a tool in place?
  - use `docs/skills/structure/skill-068-rename-object.md`
- Need to enable or disable a tool?
  - use `docs/skills/structure/skill-family-disabled-state-mutation.md`
## Boundary Rules
- Prefer these generic tool lifecycle leaves for pure operation-only prompts even when a broader class-specific lifecycle card also mentions copy/move/delete.
- Keep class-specific lifecycle cards for create/update/configuration semantics and for mixed branches where the lifecycle card is already the active owner.
- Do not route these operations through local YAML editing.
