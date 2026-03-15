# Skill Card 006: Safe Local YAML Edit Fallback (Composite)
## 1) Skill Name
Safe local YAML edit fallback (rollback-preserving composite)
## 2) Objective
Provide a reusable rollback-preserving local YAML edit workflow for file-backed assets when a selected owning leaf explicitly authorizes local YAML editing.
This card is helper-only. It is not a substitute for missing API-backed lifecycle, rename, disabled-state, or configuration coverage.
## 3) Composition Type
- Composite skill (orchestration only)
- This card does not own the semantic YAML mutation itself; the owning leaf skill still defines exactly what to edit and why local YAML is allowed.
- This card is not a first-choice routing destination for ordinary mutation prompts.
## 4) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md` when the target file path is unresolved
- Additive:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` only when the same branch also needs API interaction before or after the local YAML edit
## 5) Inputs
- Required:
  - `targetFilePath`
  - `editOwner` (the leaf skill or branch that owns the semantic YAML mutation)
- Optional:
  - `workingCopyPath`
  - `rollbackCopyPath`
  - `cleanupRollbackCopyOnSuccess`
## 6) Procedure
1. Resolve the exact local target path.
2. Confirm that the owning leaf explicitly authorizes local YAML editing for this operation.
3. Create a local rollback copy before editing.
4. If needed, create a separate working copy so the original file is not edited in place until the branch is ready to commit the change.
5. Hand off to the owning leaf skill to perform the scoped local YAML edit and local before/after verification.
6. Verify that only the intended local target block changed.
7. If verification fails, restore immediately from the rollback copy.
8. If the branch succeeds and cleanup was requested, remove the rollback copy; otherwise preserve it and report its path.
9. If the broader branch later needs API interaction, enter Skill 050 at that branch transition rather than treating this card as an API wrapper.
## 7) Validation
- An explicit rollback source exists before the first local write.
- The owning leaf skill completed scoped before/after verification.
- The edit stayed within the owning leaf's explicitly authorized YAML boundary.
- If restoration was required, the restored local file matches the rollback source.
## 8) Failure Handling
- If rollback copy creation fails: stop before editing.
- If the caller cannot name a validated owning leaf for the YAML edit: stop before editing.
- If scoped verification fails: restore immediately.
- If restore fails: stop and report the surviving rollback sources for manual recovery.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - keep a local rollback copy before editing
  - restore from that copy immediately if the branch fails verification
## 10) Use Cases
- Local YAML edits to `.tst` assets when a leaf skill explicitly authorizes them
- Future `.pva` / `.pvn` local YAML fallback branches when explicitly authorized
- Surgical exception paths such as datasource-type-targeted structural repair or constrained-client structural promotion that already have a validated owning leaf
## 11) Context Window Guidance
Load only:
- this card
- Skill 001 when the path is unresolved
- the owning leaf skill that defines the semantic YAML edit
- Skill 050 only if the branch later requires API interaction
