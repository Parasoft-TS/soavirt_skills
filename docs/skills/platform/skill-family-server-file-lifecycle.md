# Skill Family: Server File Lifecycle

Purpose: define the shared routing, safety, and verification model for file-level server operations across SOAtest and Virtualize assets.

## Scope
This family covers file-level operations on server-hosted assets such as:
- `.tst`
- `.pva`
- `.pvn`

It does not own endpoint execution details; atomic cards own the exact request/response behavior.

## Design Rule
- Keep one family card for file-lifecycle selection and anti-duplication guidance.
- Keep endpoint behavior in atomic cards.
- Use composites only for repeated rollback-preserving workflows that coordinate multiple atomic cards.

## Foundational Skills
- `docs/skills/platform/skill-001-shared-introspection.md`
  - root-aware target discovery and verification
- `docs/skills/platform/skill-002-shared-file-transfer.md`
  - download/upload and format-safe file transfer
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - mandatory preflight for write-capable branches

## Atomic Skill Matrix
### A) Core file-lifecycle operations
1. `docs/skills/platform/skill-003-server-copy.md`
   - create a new server-side copy, optionally with a destination name
2. `docs/skills/platform/skill-004-server-rename.md`
   - rename an existing file in place
3. `docs/skills/platform/skill-005-server-delete.md`
   - delete a file or explicitly targeted folder id

### B) Composite file-edit workflow
1. `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
   - use when a write workflow requires local YAML fallback with an explicit rollback-preserving envelope

## Selection Guide
- Need a new artifact or a server-side failsafe copy before riskier work?
  - use Skill 003
- Need to change only the existing file name in place?
  - use Skill 004
- Need to remove a file or cleanup artifact?
  - use Skill 005
- Need to download, edit locally, upload, verify, and restore/cleanup if needed?
  - use Skill 006 plus the owning leaf skill for the semantic edit itself

## Shared Validation Expectations
Every file-lifecycle branch should:
1. Resolve exact rooted ids under the canonical server roots before mutation.
2. Apply Skill 050 before the first write in the branch.
3. Perform deterministic post-condition verification:
   - listing/readback for copy/rename/delete
   - re-download/readback for local-edit fallback branches
4. Keep rollback handling explicit before destructive or high-risk changes.

## Shared Root Awareness
- Treat these as the canonical server root directories:
  - `/TestAssets`
  - `/VirtualAssets`
  - `/ProvisioningAssets`
- Prefer likely-root-first resolution before broader recursive scans.

## Duplication Rule
- Do not repeat shared file-lifecycle selection guidance in Skills 003-005.
- Do not use Skill 006 as a generic wrapper for all file operations; reserve it for rollback-preserving local YAML edit workflows.
