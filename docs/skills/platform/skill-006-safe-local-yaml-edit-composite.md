# Skill Card 006: Safe Local YAML Edit Fallback (Composite)

## 1) Skill Name
Safe local YAML edit fallback (rollback-preserving composite)

## 2) Objective
- Provide a reusable rollback-preserving local YAML edit workflow for server-hosted assets when a selected leaf skill must fall back to download/edit/upload behavior.
- Coordinate safe fallback-copy creation, local rollback preservation, upload/readback verification, and restore/cleanup handling.
- This card does not own the semantic YAML mutation itself; the owning leaf skill still defines exactly what to edit.

## 3) Composition Type
- Composite skill (orchestration only).
- This card must not duplicate endpoint implementation details from its atomic dependencies.
- This card must not replace the owning leaf skill's edit-specific scope rules or local diff checks.

## 4) Dependencies
- Required:
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
  - `docs/skills/platform/skill-003-server-copy.md`
  - `docs/skills/platform/skill-005-server-delete.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`

## 5) Inputs
- Required:
  - `targetFileId`
  - `workingLocalPath` (local working path, typically under `work/`)
  - `editOwner` (the leaf skill or branch that owns the semantic YAML mutation)
- Optional:
  - `fallbackCopyName` (server-side rollback copy name when a server fallback artifact should be preserved)
  - `cleanupFallbackCopyOnSuccess` (boolean)
  - `preserveServerFallbackCopy` (boolean, default true for high-risk edit branches)

## 6) Procedure
1. Resolve the exact target file id and apply Skill 050 before the first write in the branch.
2. If the branch requires a server-side fallback artifact, create a fallback copy with Skill 003 and record the fallback id.
3. Download the target file with Skill 002 and preserve the original downloaded file as the primary local rollback source.
4. Hand off to the owning leaf skill to perform the scoped local YAML edit and local before/after verification.
5. Upload the edited file with Skill 002 using the format and encoding rules required by the owning leaf skill.
6. Re-download the target file and let the owning leaf skill verify that only the intended changes persisted.
7. If verification or focused runtime validation fails:
  - restore the original target state by re-uploading the preserved original local file, or
  - if that rollback source is unavailable or unsuitable, restore from the server-side fallback copy by downloading the fallback artifact and uploading it back to the original target id
  - re-download and verify restoration.
8. If the branch succeeds and `cleanupFallbackCopyOnSuccess=true`, delete the fallback copy with Skill 005; otherwise preserve the fallback copy and report its id for later cleanup.
9. Record the operation summary:
  - target file id
  - fallback copy id (if created)
  - local rollback path
  - restore status
  - cleanup status

## 7) Validation
- Confirm an explicit rollback source exists before the first upload.
- If a server-side fallback copy was requested, confirm its id exists before local mutation proceeds.
- Confirm upload/readback verification is completed by the owning leaf skill.
- If restoration was required, confirm the restored target state by re-download/readback.
- If cleanup was requested, confirm the fallback copy no longer exists.

## 8) Failure Handling
- If fallback copy creation fails: stop before local edit/upload.
- If upload or post-upload verification fails: restore immediately before any cleanup.
- If restore fails: stop and report the surviving rollback sources and affected ids for manual recovery.
- If cleanup delete fails after a successful branch: report the orphaned fallback copy id for follow-up.

## 9) Safety / Rollback
- Read-only by default? No.
- Rollback approach:
  - Preferred: keep the original downloaded file as the primary rollback source.
  - When configured, also keep a server-side fallback copy so the original state can be reconstructed even if the local working file becomes unusable.
  - Use Skill 005 only for cleanup, not as the primary restore mechanism.

## 10) Use Cases
- Safe YAML fallback branch for constrained REST Client request/resource edits.
- YAML fallback for ambiguous structural mutation branches that need an explicit restore path.
- Future local YAML edit workflows that need shared rollback/copy/upload/cleanup coordination.

## 11) Context Window Guidance
- For this workflow, load only:
  - This card and direct dependencies (`002`, `003`, `005`)
  - the owning leaf skill that defines the semantic YAML edit
  - optionally `001` for verification convenience
- Do not use this card as a substitute for the owning leaf skill's exact edit envelope.
