# Skill Card 002: Legacy Server File Transfer (Deferred Compatibility)
## 1) Skill Name
Legacy server file transfer (`download` / `upload`) compatibility reference
## 2) Status
Deferred / legacy. Retained temporarily for compatibility and historical endpoint reference.
## 3) Objective
Document the old server-mediated file transfer path for exceptional cases where a workflow explicitly requires API byte transfer. This is not the normal path in the merged local-workspace model.
## 4) When to Use
Use this card only when:
- a task explicitly requires API file byte transfer
- local filesystem access is not the intended or sufficient path
- the branch has already determined that it is leaving local-first file work
Do not use this card for normal `.tst`, `.pva`, `.pvn`, env-file, or project-file work in this repo.
## 5) Preferred Alternatives
Prefer:
- `docs/skills/platform/skill-family-server-file-lifecycle.md`
- `docs/skills/platform/skill-001-shared-introspection.md`
- `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 6) Scope
- In scope:
  - legacy reference for `GET /v6/files/download`
  - legacy reference for `POST /v6/files/upload`
  - byte-level transfer safety notes that may still matter in rare compatibility branches
- Out of scope:
  - default file targeting
  - default file editing workflow
  - default operator-facing local asset work
  - any claim that server transfer is the standard route
## 7) Legacy Endpoint Notes
- Download endpoint: `GET /v6/files/download?id=<id>`
- Upload endpoint: `POST /v6/files/upload?id=<id>&replace=true`
- If ever used for YAML, preserve UTF-8 without BOM.
## 8) Safety Notes
- Enter Skill 050 before any write-capable API transfer branch.
- Keep a rollback source before upload.
- Re-read and verify the postcondition if this branch is used at all.
## 9) Routing Note
This card is not a primary routing destination. Load it only when another workflow explicitly calls for a legacy API-transfer compatibility branch.
## 10) Retirement Note
Remove this card once no active workflow depends on it and no meaningful legacy API-transfer branch remains.
