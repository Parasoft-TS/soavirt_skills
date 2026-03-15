# Skill Card 005: Legacy Server-Side File Delete (Deferred Compatibility)
## 1) Skill Name
Legacy server-side file delete compatibility reference
## 2) Status
Deferred / legacy. Retained temporarily for compatibility and historical endpoint reference.
## 3) Objective
Document the old server-mediated file delete path for exceptional cases where a workflow explicitly requires API-level delete rather than normal local filesystem delete.
## 4) When to Use
Use this card only when:
- a task explicitly requires the API delete route, and
- local filesystem delete is not the intended or sufficient path, and
- the branch has already determined that it is leaving local-first file work.
## 5) Preferred Alternatives
Prefer:
- local filesystem delete under `docs/skills/platform/skill-family-server-file-lifecycle.md`
- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 6) Scope
- In scope:
  - legacy reference for `DELETE /v6/files`
  - safety notes for rare compatibility branches that still need API delete semantics
- Out of scope:
  - normal delete workflow for merged-workspace assets
  - any claim that API delete is the default route
## 7) Legacy Endpoint Notes
- Historical delete path used `DELETE /v6/files?id=<id>`.
## 8) Safety Notes
- Enter Skill 050 before any write-capable API delete branch.
- Verify target absence deterministically if this branch is used.
## 9) Routing Note
This card is not a primary routing destination. Load it only for an explicit legacy compatibility branch.
## 10) Retirement Note
Remove this card once no active workflow depends on it and no meaningful legacy API-delete branch remains.
