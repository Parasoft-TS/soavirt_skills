# Skill 062: Penetration Testing Tool Workflow

## 1) Skill Name
Create, read, and update a SOAtest Penetration Testing Tool under the correct REST Client security-testing output.

## 2) Objective
Provide an endpoint-accurate lifecycle for SOAtest Penetration Testing Tools using `/v6/tools/penTestTools`, with deterministic parent selection under a REST Client `Traffic Object`, read-merge-write safety for updates, and explicit non-goal protection against attaching normal business-validation tools to the same security branch.

## 3) Scope
- In scope:
  - create Penetration Testing Tool via `POST /v6/tools/penTestTools`
  - read Penetration Testing Tool via `GET /v6/tools/penTestTools?id=...`
  - update Penetration Testing Tool via `PUT /v6/tools/penTestTools?id=...`
  - deterministic parent selection under REST Client `Traffic Object`
  - readback verification of id, parent relationship, and persisted settings
- Out of scope:
  - deciding whether security coverage is in scope for a workflow
  - copying the happy-path REST Client or scenario subtree into a security suite before tool creation
  - choosing single-step vs minimal-prefix vs full-sequence derivation for the branch
  - performing operation-level security breadth/accounting or choosing the lifecycle-preferred seed when the caller has several credible copies of the same operation
  - chaining validator/assertor/diff tools onto the same security branch
  - treating `Traffic Object` as a default business-validation parent

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`

## 4) Inputs
- Required for create:
  - target REST Client id, or resolved `Traffic Object` output-provider parent id
- Optional for create/update:
  - tool name
  - `toolSettings`
- Required for read/update:
  - Penetration Testing Tool id

## 5) Preconditions
- API reachable and authenticated.
- The target REST Client or output-provider parent is resolvable.
- Security coverage has already been approved by the calling workflow when this tool is being created as part of an orchestration branch.

## 6) Procedure
### 6.1 Capability and Parent Resolution
1. Before the first write, run branch-aware capability preflight (Skill 050).
2. Resolve the correct parent id using Skill 018.
3. Valid parent rule:
   - if the caller supplies a REST Client id, construct the canonical parent id as `<rest-client-id>/Traffic Object`
   - if the caller supplies an output-provider id directly, verify that it is the REST Client `Traffic Object` output-provider path
4. Do not guess or infer other request-side or diagnostics-side parents.

### 6.2 Create
5. Build the create payload for `POST /v6/tools/penTestTools`:
   - `parent.id` = resolved REST Client `Traffic Object`
   - optional `name`
   - optional `toolSettings`
6. Execute `POST /v6/tools/penTestTools`.
7. Read back the created tool with `GET /v6/tools/penTestTools?id=<created-id>`.
8. Verify:
   - returned id is stable
   - `relationships.parentRel.id` points at the intended REST Client `Traffic Object` parent
   - intended `name` and `toolSettings` persisted

### 6.3 Read
9. Use `GET /v6/tools/penTestTools?id=<id>`.
10. Confirm the tool remains attached to the intended REST Client `Traffic Object` parent before making assumptions about the security branch structure.

### 6.4 Update
11. Use read-merge-write safety for `PUT /v6/tools/penTestTools?id=<id>`.
12. Read the current tool first.
13. Merge only the intended changes into the full payload shape.
14. Execute `PUT /v6/tools/penTestTools?id=<id>`.
15. Read back again and verify the updated settings persisted.

### 6.5 Payload-Shape Usage Guard (Disambiguation Only)
The example payloads below are shape references, not semantic defaults.

Rules:
- Replace all placeholders from caller-approved parent/name/settings intent.
- Create payloads use `parent.id`; update payloads do not invent create-only wrappers or relocation intent.
- For updates, start from live GET readback and mutate only the intended fields before PUT.
- Do not guess alternate response-side or request-side parent paths; the only approved parent for this skill is the REST Client `Traffic Object`.

Minimal create shape:
```json
{
  "parent": {
    "id": "<rest-client-id>/Traffic Object"
  },
  "name": "<pen-test-name>",
  "toolSettings": {}
}
```

Minimal update shape:
```json
{
  "name": "<pen-test-name>",
  "toolSettings": {}
}
```

## 7) Canonical Endpoint Notes
- `POST /v6/tools/penTestTools`
  - request schema: `penTestToolsRequest`
  - required field: `parent`
- `GET /v6/tools/penTestTools?id=...`
  - response schema: `penTestToolsResponse`
- `PUT /v6/tools/penTestTools?id=...`
  - request schema: `penTestToolsPutRequest`
  - overwrite semantics apply when fields are omitted, so use read-merge-write

## 8) Validation
- Create/read/update requests resolve through the `penTestTools` endpoints rather than ad hoc tool-family guesses.
- Parent selection uses REST Client `Traffic Object` exactly.
- The created/updated tool is read back successfully.
- The persisted parent relationship still points at the intended security branch REST Client `Traffic Object`.
- The skill does not attach validator/assertor/diff tools as part of the same workflow.

## 9) Failure Modes
- Penetration Testing Tool is attached directly under the REST Client id instead of the `Traffic Object` output-provider.
- `Traffic Object` is treated as a generic business-validation parent.
- Sparse `PUT` drops settings because overwrite semantics were ignored.
- Create-shape vs update-shape confusion can introduce unsupported parent/move semantics or silently discard intended settings.
- Security-tool creation is followed by invalid extra chaining of validator/assertor/diff tools.

## 10) Safety / Rollback
- This is a write-capable tool-lifecycle skill.
- Rollback options:
  - update back to the last known-good configuration via read-merge-write, or
  - delete/recreate through the calling workflow when that workflow owns the surrounding security branch lifecycle.

## 11) Reuse Notes
- Primary target: SOAtest.
- Typical callers: Skill 033 and upgraded Skill 065 after the calling workflow has already performed any required security accounting and copied the intended exact-operation REST Client or minimal scenario subtree into a security branch.
- Treat this skill as the canonical exception that authorizes REST Client `Traffic Object` chaining for security testing.
