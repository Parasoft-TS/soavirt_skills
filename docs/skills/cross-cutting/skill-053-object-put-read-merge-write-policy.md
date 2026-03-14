# Skill 053: Object PUT Read-Merge-Write Policy (Suites and Non-Tool Objects)

## 1) Skill Name
Safe update policy for suite/non-tool object configurations using GET -> mutate -> PUT.

## 2) Objective
Prevent accidental configuration loss when editing suites and other non-tool objects by requiring read-merge-write updates instead of sparse PUT payloads.

## 3) Scope
- In scope:
  - updating existing test suites via `PUT /v6/suites/testSuites?id=...`
  - updating other non-tool objects that expose replace-semantics PUT behavior
  - preserving current configuration fields not intentionally changed
- Out of scope:
  - creating new objects (`POST` workflows)
  - class-specific tool updates (covered by Skill 049)

## 4) Inputs
- Required:
  - target object id
  - intended field-level mutations
- Optional:
  - allowlist of expected changed fields
  - rollback snapshot location

## 5) Preconditions
- API reachable and authenticated.
- Target object exists and is readable by class-specific `GET`.
- Update target is an existing object (not create flow).

## 6) Procedure
1. Read current object config via class-specific `GET` (for suites: `GET /v6/suites/testSuites?id=...`).
2. Build merged PUT payload:
  - start from current object fields accepted by endpoint PUT schema,
  - apply only intended mutations,
  - preserve all other fields as-is.
3. Execute class-specific `PUT` with merged payload.
4. Read back same object and verify:
  - intended fields changed,
  - unrelated fields preserved.
5. If verification fails:
  - stop downstream writes,
  - restore from pre-update snapshot.

## 7) Validation
- Required checks:
  - intended mutations are present after PUT,
  - non-target fields do not regress,
  - critical execution/configuration fields remain intact.

## 8) Failure Modes
- Sparse PUT overwrite: omitted fields revert to defaults.
- Endpoint schema drift: payload field mismatch or incompatible shape.
- Silent partial regression: update appears successful but non-target fields are reset.

## 9) Safety / Rollback
- Treat every non-tool PUT as destructive-risk unless proven safe.
- Keep pre-update snapshot (`GET` response) for immediate rollback.

## 10) Reuse Notes
- Primary target: SOAtest.
- Also intended as a shared policy surface for other product object-update flows that expose replace-semantics `PUT` behavior.
- Composition rule:
  - load before any suite/non-tool update skill that performs PUT edits.
  - pair with Skill 050 preflight for endpoint-method compatibility.

## 11) Related Policies
- `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` (tool objects)
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` (compatibility preflight)
