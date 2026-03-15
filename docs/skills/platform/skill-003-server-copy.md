# Skill Card 003: Legacy Server-Side File Copy (Deferred Compatibility)
## 1) Skill Name
Legacy server-side file copy compatibility reference
## 2) Status
Deferred / legacy. Retained temporarily for compatibility and historical endpoint reference.
## 3) Objective
Document the old server-mediated file-copy path for exceptional cases where a workflow explicitly requires API-level file duplication rather than normal local filesystem copy.
## 4) When to Use
Use this card only when:
- a task explicitly requires `POST /v6/files/copy`, and
- local filesystem copy is not the intended or sufficient path, and
- the branch has already determined that it is leaving local-first file work.
## 5) Preferred Alternatives
Prefer:
- local filesystem copy under `docs/skills/platform/skill-family-server-file-lifecycle.md`
- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 6) Scope
- In scope:
  - legacy reference for `POST /v6/files/copy`
  - safety notes for rare compatibility branches that still need server-side copy semantics
- Out of scope:
  - normal copy workflow for merged-workspace assets
  - copied-target naming strategy for current operator-facing workflows
  - any claim that server-side copy is the default route
## 7) Legacy Endpoint Notes
- Copy endpoint: `POST /v6/files/copy`
- Historical request shape used `from.id`, `to.parent.id`, and optional `to.name`.
## 8) Safety Notes
- Enter Skill 050 before any write-capable API copy branch.
- Verify the copied target deterministically if this branch is used.
## 9) Routing Note
This card is not a primary routing destination. Load it only for an explicit legacy compatibility branch.
## 10) Retirement Note
Remove this card once no active workflow depends on it and no meaningful legacy API-copy branch remains.
