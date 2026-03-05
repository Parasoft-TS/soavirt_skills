# Skill 024: Create TST from RAML

## 1) Skill Name
Create a new SOAtest `.tst` from RAML.

## 2) Objective
Create a brand-new `.tst` with generated tests from a RAML definition using `POST /v6/files/tsts/raml`.

## 3) Scope
- In scope:
  - create `.tst` from RAML URL or file-system path
  - verify generated file/root suite
  - optional environment creation object
  - optional cleanup delete
- Out of scope:
  - traffic-based generation
  - post-generation customization

## 4) Inputs
- Required:
  - `parent.id` (directory id, for example `/TestAssets`)
  - `name` (file base name)
- Conditional required (must be resolved before create):
  - RAML source location (user-provided URL or file-system path)
- Optional:
  - `createEnvironment`

## 4.1) Required Input Resolution Rule
- Resolve RAML source input using this precedence:
  1. explicit value in current user request,
  2. value confirmed earlier in current session,
  3. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
- Apply confidence gate before create:
  - proceed only when one candidate is unambiguous and format-compatible with RAML,
  - if source is missing, conflicting, or weakly inferred, ask a targeted user question and wait for confirmation,
  - do not call `POST /v6/files/tsts/raml` without a confirmed source location.

## 5) Preconditions
- API reachable and authenticated.
- Parent directory exists and is writable.
- RAML source is reachable and parseable from server runtime.

## 5.1) Service-Definition Source Readiness Preflight
Run this preflight before `POST /v6/files/tsts/raml`:
1. Confirm source location mode (URL vs file path).
2. For URL mode, probe the exact RAML URL and expect `200` from server runtime.
3. Confirm content matches endpoint expectation (RAML for this skill).
4. Confirm `parent.id` exists and is writable.
5. If preflight fails, correct source/parent and retry.

## 6) Procedure
1. Resolve required RAML location from user-provided URL or file path using the input-resolution rule.
2. If location is not confidently resolved, ask user for RAML source and stop before write.
3. Build payload with parent/name/location.
4. `POST /v6/files/tsts/raml`.
5. Validate returned `id`, `url`, and root `Test Suite` relation.
6. Optional rollback: `DELETE /v6/files?id=<created-id>`.

## 7) Canonical Payload (API-First)
```json
{
  "parent": { "id": "/TestAssets" },
  "name": "My_RAML_Test",
  "location": { "url": "http://example.local/api.raml" }
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
- `404 location.url could not be found` when source URL is not accessible.
- Parser/format errors when RAML content is invalid for generator.
- Source readiness mismatch: URL accessible externally but not resolvable from server runtime.
- Missing or ambiguous RAML source location when write is attempted.

## 10) Safety / Rollback
- Write operation; rollback by deleting created `.tst`.

## 11) Reuse Notes
- Applies to SOAtest: Endpoint contract validated from OpenAPI spec.
- API-first authoring required; no dependency on pre-existing generated examples.

## 12) Current Validation Status (2026-03-03)
- Contract validated:
  - endpoint exists: `POST /v6/files/tsts/raml`
  - request schema: `tstsRamlRequest` (`parent`, `name`, `location` required)
- Runtime validation pending a reachable live RAML source.
