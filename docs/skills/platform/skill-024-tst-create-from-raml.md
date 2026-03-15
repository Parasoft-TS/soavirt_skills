# Skill 024: Create TST from RAML

## 1) Skill Name
Create a new SOAtest `.tst` from RAML.

## 2) Objective
Create a brand-new `.tst` with generated tests from a RAML definition using `POST /v6/files/tsts/raml`.

## 3) Scope
- In scope:
  - create `.tst` from RAML URL or file-system path
  - verify generated file/root suite
  - deterministic handling of the optional environment-creation request
  - optional cleanup delete
- Out of scope:
  - traffic-based generation
  - post-generation customization

## 3.2) Routing Boundary (Required)
- Use this card as a direct route only when the user explicitly requests creating a **new `.tst`** from a RAML source.
- Do not use this card as the first route for help-style or outcome-level prompts when output mode is not explicit.
- For those ambiguous prompts, route to `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` first.

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md` (post-create traceability/tagging handoff)
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`

## 4) Inputs
- Required:
  - `parent.id` (default to the active project root when a project is already active and the user did not override it; for example `/TestAssets/<Project>` or `/TestAssets`)
  - `name` (file base name)
- Conditional required (must be resolved before create):
  - RAML source location (user-provided URL or file-system path)
- Optional:
  - `restCreateEnvironment` object, but only when the environment branch was already explicitly resolved using Skill 064 generation modes (`disabled`, `local_managed`, `reference_external`)

## 4.1) Required Input Resolution Rule
- Resolve RAML source input using this precedence:
  1. explicit value in current user request,
  2. active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
  3. value confirmed earlier in current session,
  4. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
- When a project is already active and `parent.id` was not explicitly chosen, default the new `.tst` to that project's root test-assets directory rather than a repo-global catch-all folder.
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
1. Apply Skill 050 Profile B for `.tst` generation create/readback compatibility.
2. Confirm source location mode (URL vs file path).
3. For URL mode, probe the exact RAML URL and expect `200` from server runtime.
4. Confirm content matches endpoint expectation (RAML for this skill).
5. If preflight fails, correct source/parent and retry.

## 6) Procedure
1. Resolve required RAML location from user-provided URL or file path using the input-resolution rule.
2. If location is not confidently resolved, ask user for RAML source and stop before write.
3. Build payload with the resolved project-aware parent, name, and location:
   - default v1 branch: omit `restCreateEnvironment`, rely on the generator's built-in local environment behavior, and do not plan a separate post-create environment creation step
   - include `restCreateEnvironment` only when the current request or already-confirmed upstream context explicitly resolves a non-default or explicitly parameterized environment branch from Skill 064
4. `POST /v6/files/tsts/raml`.
5. Validate returned `id`, `url`, and root `Test Suite` relation, then inspect the generated `.tst` environment state and reuse it as the initial environment context rather than creating a second environment by default.
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
- Primary target: SOAtest.
- Covers the `POST /v6/files/tsts/raml` generation path for RAML source inputs.
- API-first authoring required; no dependency on pre-existing generated examples.
- If requirements traceability or tagging is requested, run Skill 009 on the root test suite immediately after creation.
- Use Skill 064 for `restCreateEnvironment` selection and any follow-on environment mechanics; v1 default generation mode is `local_managed`, which for brand-new `.tst` generation means relying on the generator's built-in local environment behavior unless the user or already-confirmed upstream context explicitly selects another supported branch.
