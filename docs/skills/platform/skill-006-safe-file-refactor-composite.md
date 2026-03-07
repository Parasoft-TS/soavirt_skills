# Skill Card 006: Safe File Refactor (Composite)

## 1) Skill Name
Safe File Refactor (copy → optional rename → verify → optional cleanup)

## 2) Objective
- Perform low-risk server-side file manipulations using validated atomic skills while preserving rollback options.

## 3) Composition Type
- Composite skill (orchestration only).
- This card **must not** duplicate endpoint implementation details from atomic cards.

## 4) Dependencies
- `skill-003-server-copy-rename.md` (copy with destination naming)
- `skill-004-server-rename.md` (in-place rename)
- `skill-005-server-delete.md` (cleanup/remove temporary files)
- Optional verification support: `skill-001-shared-introspection.md`
- Mandatory guard:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`

## 5) Inputs
- Required:
  - `sourceId`
  - `workingName` (temporary or target copy name)
  - `parentId` (destination folder)
- Optional:
  - `finalName` (if different from `workingName`)
  - `cleanup` (boolean: delete temporary working copy)

## 6) Procedure
1. Create a working copy from `sourceId` using Skill 003.
2. Verify working copy exists in destination listing.
3. If `finalName` differs, rename in place using Skill 004.
4. Verify final id exists and previous id does not.
5. If `cleanup=true` for temporary artifacts, delete temporary id with Skill 005.
6. Record operation summary and resulting ids.

## 7) Validation
- Confirm each atomic step returns success and is verified in listing.
- Ensure original source file remains unchanged unless explicitly targeted.
- Ensure final expected id exists exactly once.

## 8) Failure Handling
- If copy fails: stop; do not run rename/delete.
- If rename fails after successful copy: keep working copy and report for manual review.
- If cleanup delete fails: report orphaned temp id for follow-up.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback approach:
  - Preferred: preserve original source; perform operations on copied working file.
  - If final state is undesired, delete copied/refactored artifact via Skill 005.

## 10) Use Cases
- Create safe branch-like test assets for experimentation.
- Rename staging assets before promotion.
- Prepare isolated copies before deeper YAML configuration edits.

## 11) Context Window Guidance
- For this workflow, load only:
  - This card (`skill-006`) and direct dependencies (`003`, `004`, `005`)
  - Optionally `001` for listing/verification convenience
- Avoid loading YAML edit cards unless content changes are required.
