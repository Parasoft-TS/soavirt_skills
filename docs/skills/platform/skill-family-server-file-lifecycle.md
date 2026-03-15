# Skill Family: Local Workspace File Lifecycle
Purpose: define the shared local-first selection, safety, and verification model for file-level operations across merged-workspace SOAtest and Virtualize assets.
## Scope
This family covers file-backed asset operations under the fixed merged-workspace roots:
- `TestAssets/`
- `VirtualAssets/`
- `ProvisioningAssets/`
It owns local-first selection guidance, not endpoint-level API semantics.
## Design Rule
- Keep one family card for local file-lifecycle selection and anti-duplication guidance.
- Keep local path-resolution discipline in Skill 001.
- Keep rollback-preserving local YAML edit choreography in Skill 006.
- Keep API-branch discipline in Skill 050.
## Foundational Skills
- `docs/skills/platform/skill-001-shared-introspection.md`
  - local asset-path resolution and ambiguity handling
- `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - rollback-preserving local YAML edit envelope
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - API-branch preflight when a branch leaves local-only work
## Selection Guide
- Need to resolve where a file-backed asset lives in the merged workspace?
  - use Skill 001
- Need to read, copy, rename, or delete a file-backed asset after the target path is known?
  - use local filesystem operations directly under this family's rules
- Need to edit YAML locally with an explicit rollback envelope?
  - use Skill 006 plus the owning leaf skill
- Need to interact with the localhost API for runtime-object ids, semantic mutation, generation, execution, or diagnostics?
  - enter Skill 050 at the API branch transition
## Local-First Rules
- Explicit local paths win immediately.
- When the path is not explicit, search the active project first when project context exists.
- Use likely-root-first search before broadening across other roots/projects.
- Keep file-backed path identity separate from API-authoritative runtime object identity.
- Fail closed and ask for clarification when multiple plausible local targets remain.
## Shared Validation Expectations
Every local file-lifecycle branch should:
1. confirm the exact local target before mutation
2. keep operations scoped to the merged asset roots
3. verify the intended postcondition deterministically after the operation
4. avoid broad destructive guessing when multiple plausible targets exist
## Escalation Rule
Escalate out of this family when:
- runtime object ids are required
- semantic API mutation or generation is required
- execution or diagnostics are required
- a branch explicitly authorizes local YAML editing and therefore needs Skill 006
## Legacy Transitional Cards
The old server-era transfer/copy/rename/delete cards (`002-005`) may remain temporarily in the repo for downstream compatibility cleanup, but they are no longer the primary operator-facing route in the merged local-workspace model.
