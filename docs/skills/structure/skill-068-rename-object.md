# Skill 068: Rename Object
## 1) Skill Name
Rename one supported suite or tool in place through its owning class-specific API surface.
## 2) Objective
Provide one atomic API-backed owner for ordinary rename requests so agents stop treating local YAML editing as the default way to rename runtime objects inside a `.tst`.
## 3) Scope
- In scope:
  - in-place rename of one existing test suite via `GET/PUT/GET /v6/suites/testSuites?id=...`
  - in-place rename of one existing responder suite via `GET/PUT/GET /v6/suites/responderSuites?id=...`
  - in-place rename of one existing supported tool via its class-specific `GET/PUT/GET` surface
  - class detection for tools from the resolved object URL or generic `GET /v6/tools?id=...` readback
- Supported tool classes in v1:
  - DB Tool
  - REST Client
  - SOAP Client
  - JSON Assertor
  - XML Assertor
  - JSON Validator
  - XML Validator
  - Diff Tool
  - JSON Data Bank
  - XML Data Bank
  - Penetration Testing Tool
- Out of scope:
  - local filesystem rename of the `.tst` file itself
  - broad configuration edits beyond the `name` field
  - unsupported object classes where the class-specific rename branch is not yet codified
  - YAML fallback for ordinary rename work
## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Required by branch:
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` for tool branches
  - `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md` for suite and responder-suite branches
## 4) Inputs
- Required:
  - target runtime object id
  - target object kind or enough evidence to resolve it deterministically
  - new name
- Optional:
  - expected current name
  - expected parent id for drift detection
## 5) Preconditions
- The target id resolves through the owning runtime API branch.
- Capability preflight confirms the needed class-specific `GET` and `PUT` route.
- The caller has identified or can derive the correct class-specific endpoint before mutation.
## 6) Procedure
1. Resolve the concrete target kind before mutation.
2. If the caller already has a class-specific object URL, use it as the branch selector.
3. If the caller has only a generic tool id, run `GET /v6/tools?id=<tool-id>` and use the returned `url` to determine the concrete class-specific endpoint.
4. Map the target to the owning rename surface:
   - test suite -> `/v6/suites/testSuites`
   - responder suite -> `/v6/suites/responderSuites`
   - DB Tool -> `/v6/tools/dbTools`
   - REST Client -> `/v6/tools/restClients`
   - SOAP Client -> `/v6/tools/soapClients`
   - JSON Assertor -> `/v6/tools/jsonAssertors`
   - XML Assertor -> `/v6/tools/xmlAssertors`
   - JSON Validator -> `/v6/tools/jsonValidators`
   - XML Validator -> `/v6/tools/xmlValidators`
   - Diff Tool -> `/v6/tools/diffTools`
   - JSON Data Bank -> `/v6/tools/jsonDataBanks`
   - XML Data Bank -> `/v6/tools/xmlDataBanks`
   - Penetration Testing Tool -> `/v6/tools/penTestTools`
5. Read the full current object through that class-specific `GET` endpoint.
6. Preserve the full GET payload as the mutation base.
7. Replace only the `name` field.
8. Write the full merged object back through the same class-specific `PUT` endpoint.
9. Read back through the same class-specific `GET` endpoint.
10. Confirm postconditions:
   - the object id is unchanged
   - the name now matches the requested value
   - unrelated fields remain present after readback
   - when the response exposes relationships, the parent relationship is unchanged unless the caller explicitly expected a different parent from another owning workflow
## 7) Validation
- Rename uses the class-specific endpoint, never generic `PUT /v6/tools`.
- The same object id survives the rename.
- The readback name matches the requested new name.
- The class-specific payload remains intact after the PUT.
## 8) Failure Modes
- Using generic `PUT /v6/tools` for rename even though it only owns `disabled`.
- Sending a sparse PUT that drops unrelated object configuration.
- Falling back to YAML for ordinary rename work.
- Renaming the wrong class because generic tool URL/type evidence was not resolved first.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - preserve the full pre-change GET payload
  - restore through the same class-specific `PUT` path if postcondition verification fails after a partial rename
## 10) Reuse Notes
- Use this card for pure rename intent or rename-only substeps inside a broader approved plan.
- If the branch also needs broader suite/tool configuration edits, the owning lifecycle card may still perform rename as part of its read-merge-write update, but narrow rename-only prompts should route here first.
