# Skill 023: Create TST from WSDL

## 1) Skill Name
Create a new SOAtest `.tst` from WSDL.

## 2) Objective
Create a brand-new `.tst` with generated tests from a WSDL definition using `POST /v6/files/tsts/wsdl`.

## 3) Scope
- In scope:
  - create `.tst` from WSDL URL or file-system path
  - verify generated file/root suite
  - optional environment creation object
  - optional cleanup delete
- Out of scope:
  - traffic-based generation
  - post-generation customization

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md` (post-create traceability/tagging handoff)

## 4) Inputs
- Required:
  - `parent.id` (directory id, for example `/TestAssets`)
  - `name` (file base name)
- Conditional required (must be resolved before create):
  - WSDL source location (user-provided URL or file-system path)
- Optional:
  - `createEnvironment`

## 4.1) Required Input Resolution Rule
- Resolve WSDL source input using this precedence:
  1. explicit value in current user request,
  2. value confirmed earlier in current session,
  3. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
- Apply confidence gate before create:
  - proceed only when one candidate is unambiguous and format-compatible with WSDL,
  - if source is missing, conflicting, or weakly inferred, ask a targeted user question and wait for confirmation,
  - do not call `POST /v6/files/tsts/wsdl` without a confirmed source location.

## 5) Preconditions
- API reachable and authenticated.
- Parent directory exists and is writable.
- WSDL source is reachable and parseable from server runtime.

## 5.1) Service-Definition Source Readiness Preflight
Run this preflight before `POST /v6/files/tsts/wsdl`:
1. Apply Skill 050 Profile B for `.tst` generation create/readback compatibility.
2. Confirm source location mode (URL vs file path).
3. For URL mode, probe the exact WSDL URL and expect `200` from server runtime.
4. Confirm content matches endpoint expectation (WSDL for this skill).
5. If preflight fails, correct source/parent and retry.

## 6) Procedure
1. Resolve required WSDL location from user-provided URL or file path using the input-resolution rule.
2. If location is not confidently resolved, ask user for WSDL source and stop before write.
3. Build payload with parent/name/location.
4. `POST /v6/files/tsts/wsdl`.
5. Validate returned `id`, `url`, and root `Test Suite` relation.
6. Optional rollback: `DELETE /v6/files?id=<created-id>`.

## 7) Canonical Payload (API-First)
```json
{
  "parent": { "id": "/TestAssets" },
  "name": "My_WSDL_Test",
  "location": { "url": "http://localhost:8090/parabank/services/ParaBank?wsdl" }
}
```

## 8) Validation
- Expected status:
  - `200` success
  - `400` malformed payload
  - `404` invalid parent or unresolved `location`
- Post-check:
  - created `id` ends in `.tst`
  - response includes root `Test Suite`

## 9) Failure Modes
- `404 location.url could not be found` when source URL is not accessible from server runtime.
- `400 Invalid payload` for malformed JSON/schema mismatch.
- Source readiness mismatch: URL accessible externally but not resolvable from server runtime.
- Missing or ambiguous WSDL source location when write is attempted.

## 10) Safety / Rollback
- Write operation; rollback by deleting created `.tst`.

## 11) Reuse Notes
- Primary target: SOAtest.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- API-first authoring required; do not depend on pre-existing generated examples.
- If requirements traceability or tagging is requested, run Skill 009 on the root test suite immediately after creation.
