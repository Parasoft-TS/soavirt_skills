# Skill Card 004: Legacy Server-Side File Rename (Deferred Compatibility)
## 1) Skill Name
Legacy server-side file rename compatibility reference
## 2) Status
Deferred / legacy. Retained temporarily for compatibility and historical endpoint reference.
## 3) Objective
Document the old server-mediated in-place rename path for exceptional cases where a workflow explicitly requires API-level rename rather than normal local filesystem rename.
## 4) When to Use
Use this card only when:
- a task explicitly requires the API rename route, and
- local filesystem rename is not the intended or sufficient path, and
- the branch has already determined that it is leaving local-first file work.
## 5) Preferred Alternatives
Prefer:
- local filesystem rename under `docs/skills/platform/skill-family-server-file-lifecycle.md`
- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 6) Scope
- In scope:
  - legacy reference for the API-based in-place file rename path
  - safety notes for rare compatibility branches that still need it
- Out of scope:
  - normal rename workflow for merged-workspace assets
  - any claim that API rename is the default route
## 7) Legacy Endpoint Notes
- Historical rename path used `PUT /v6/files?id=<id>` with a body containing `name`.
## 8) Safety Notes
- Enter Skill 050 before any write-capable API rename branch.
- Record the prior name so rollback is possible if this branch is used.
## 9) Routing Note
This card is not a primary routing destination. Load it only for an explicit legacy compatibility branch.
## 10) Retirement Note
Remove this card once no active workflow depends on it and no meaningful legacy API-rename branch remains.
