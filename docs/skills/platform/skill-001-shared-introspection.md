# Skill Card 001: Shared Asset Path Resolution
## 1) Skill Name
Shared asset path resolution and local discovery
## 2) Objective
Resolve file-backed assets in the merged workspace safely and deterministically using local-path authority, project-local search, likely-root-first search, and fail-closed ambiguity handling.
## 3) Scope
- In scope:
  - resolve explicit local asset paths
  - search for file-backed assets under `TestAssets/`, `VirtualAssets/`, and `ProvisioningAssets/`
  - prefer active-project-local search before broader search
  - likely-root-first search by asset family/type
  - exact-name matching before broader recursive search
  - ambiguity detection and clarification thresholds
- Out of scope:
  - API runtime-object discovery
  - downloading or uploading file bytes
  - semantic mutation of asset contents
## 3.1) Dependencies
- Required:
  - none
- Additive:
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md` when active project context is available
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` only when the branch later needs API interaction
## 4) Inputs
- Optional:
  - explicit local path
  - filename or partial path
  - file-backed asset type (`tst`, `pva`, `pvn`, env, data file, resource file)
  - active project context
## 5) Preconditions
- Workspace roots are locally readable.
## 5.1) Root Invariants
Treat these as the fixed local roots for merged-workspace asset discovery:
- `TestAssets/`
- `VirtualAssets/`
- `ProvisioningAssets/`
## 6) Procedure
1. If the caller provides an explicit local path, use it directly.
2. If active project context exists, search the canonical project-local locations first:
   - `TestAssets/<project>/`
   - `TestAssets/<project>/environments/`
   - `TestAssets/<project>/data/`
   - `TestAssets/<project>/resources/`
3. If only a filename/partial path is known, choose the likely root first:
   - `.tst` and SOAtest project files -> `TestAssets/`
   - `.pva` -> `VirtualAssets/`
   - `.pvn` -> `ProvisioningAssets/`
4. Prefer exact-name immediate-child lookup in the likely location before broader recursive search.
5. Broaden to other projects or sibling roots only after the likely-root/local search returns no match.
6. If one clear match remains, return that local path.
7. If multiple plausible matches remain, stop and ask the user to clarify.
8. If the task later requires runtime object ids rather than file-backed identity, stop local targeting and enter the API lane through Skill 050.
## 7) Validation
- The returned target is a concrete local path.
- The path exists and is readable for the intended branch.
- If multiple plausible paths were found, no guess was made.
## 8) Failure Modes
- A runtime object request is mistaken for a file-backed path-resolution request.
- Search broadens across roots/projects before exhausting the active-project/likely-root path.
- Multiple matches are silently guessed instead of clarified.
## 9) Safety / Rollback
- Read-only by default? Yes
- Rollback plan for writes: not applicable
## 10) Reuse Notes
- Use this card as the canonical owner of local asset targeting discipline.
- Do not use it to infer API-authoritative runtime object ids from local YAML alone.
