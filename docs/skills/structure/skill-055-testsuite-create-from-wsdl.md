# Skill 055: Create Test Suite from WSDL in Existing `.tst`

## 1) Skill Name
Generate a WSDL-based Test Suite inside an existing SOAtest `.tst` with safe environment normalization.

## 2) Objective
Safely generate WSDL-derived SOAP tests under an existing suite using `POST /v6/suites/testSuites/wsdl`, then normalize any generated environment state so the target `.tst` is left in a stable, reusable condition for future work.

## 3) Scope
- In scope:
  - generate a WSDL-based suite under an existing suite in an existing `.tst`
  - validate actual generated placement under the target subtree
  - inspect existing active environment before generation
  - use collision-aware variable naming for generated environment variables
  - normalize generated variables into the existing active environment when generation creates a new extra environment
  - delete the generated extra environment after successful normalization
  - restore environment state during rollback
- Out of scope:
  - new `.tst` file creation from WSDL
  - suite-level generation from OpenAPI/RAML/XSD
  - runtime execution of generated tests as a required validation step
  - treating `referenceExistingEnvironment.file` as a safe/default substitute for `local_managed` environment normalization
  - assuming the API exposes the UI "Organize as Positive and Negative Unit Tests" toggle for WSDL generation

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
  - `docs/skills/cross-cutting/skill-053-object-put-read-merge-write-policy.md`
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/platform/skill-023-tst-create-from-wsdl.md` (contrast with file-level generation)

## 4) Inputs
- Required:
  - target parent suite id inside an existing `.tst`
  - generated suite name
  - WSDL source location (URL or file-system path)
- Optional:
  - environment mode:
    - `local_managed` existing-`.tst` generation (preferred default)
    - `disabled` environment generation (hardcoded generated values)
    - `reference_external` referenced external environment file (validated, non-default)
  - preferred variable prefix override
  - `variableTypes`:
    - `wsdlUriFields`
    - `clientEndpoints`
    - `both` (preferred default)

## 4.1) Required Input Resolution Rule
- Resolve WSDL source input using this precedence:
  1. explicit value in current user request,
  2. active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
  3. value confirmed earlier in current session,
  4. relevant environment variable already configured for this target.
- Normalize source to API-compatible location fields internally:
  - URL input -> `location.url`
  - file path input -> resolve to workspace-accessible source and use `location.id` when applicable.
- Resolve target context before write:
  - read the target parent suite,
  - resolve the owning `.tst` / root suite,
  - inspect current root-suite environments and the active local environment if present.
- If environment generation is enabled for an existing `.tst`, inspect the active environment variables before finalizing the create payload, and when project context already defines canonical external environment files or generation-mode expectations, consult Skill 064 before finalizing the branch.

## 4.2) Variable-Naming Decision Rule (Required)
- Preferred default for existing-`.tst` generation is `local_managed` environment mode.
- If the active environment already contains any variable names that the requested `variableTypes` branch would create, use a deterministic prefix strategy before generation.
- If no collisions are expected, non-prefixed generation is allowed.
- If a user-supplied prefix would still collide with existing variables, resolve the collision before write; do not proceed with an ambiguous prefix.
- Do not rely on same-name environment append semantics from the API.

## 5) Preconditions
- API reachable and authenticated.
- Target parent suite exists and is writable.
- WSDL source is reachable and parseable from server runtime.
- If `local_managed` environment mode is selected:
  - the owning `.tst` is readable,
  - the active environment state can be read before mutation,
  - the active environment can be updated after generation if normalization is needed.

## 5.1) Service-Definition Source Readiness Preflight
Run this preflight before `POST /v6/suites/testSuites/wsdl`:
1. Apply Skill 050 Profile E for mutation routing plus create/readback compatibility.
2. Confirm target parent suite id and owning root suite id.
3. Confirm source location mode (URL vs file path).
4. For URL mode, probe the exact WSDL URL and expect `200` from server runtime.
5. Confirm content matches endpoint expectation (WSDL for this skill).
6. If preflight fails, stop before write and correct source/parent first.

## 5.2) Environment Baseline Snapshot (Required)
Before the first write:
1. Read the root-suite children and record the current environment ids.
2. Read the active environment and preserve its full pre-change payload.
3. Record the current target-parent child ids.
4. Use these snapshots for both post-create comparison and rollback.

## 6) Procedure
1. Resolve required WSDL location from user-provided URL or file path using the input-resolution rule.
2. Resolve the target parent suite id and owning `.tst` / root suite.
3. Inspect the current active environment and its variable inventory.
4. Choose generation mode using Skill 064 terminology:
   - if the user explicitly wants hardcoded generated values, use `disabled` and set `createEnvironment.enabled=false`;
   - if the user explicitly wants an external referenced environment file, use `reference_external`, route the external-file/reference semantics through Skill 064, and treat the branch as non-default;
   - otherwise use `local_managed` with collision-aware prefixing when needed.
5. Build the create payload:
   - `parent.id=<target-parent-suite-id>`
   - `name=<generated-suite-name>`
   - `location.url=<wsdl-url>` or `location.id=<workspace-file-id>`
   - `createEnvironment`:
     - disabled mode -> `{ "enabled": false }`
     - `local_managed` mode -> `enabled=true`, `environmentSource.addNewEnvironment.name=<active-environment-name>`, optional `prefix`, `variableTypes=<requested-or-default>`
     - `reference_external` mode -> `enabled=true`, `environmentSource.referenceExistingEnvironment.file.id=<env-file-id>`, `variableTypes=<requested-or-default>`
6. `POST /v6/suites/testSuites/wsdl`.
7. Do not trust the POST response id alone for placement, especially when the parent is nested.
8. Re-read the target parent children and target subtree descendants to locate the actual generated content.
9. Environment-normalization branch for `local_managed` mode:
   - compare the post-create environment list with the pre-change baseline;
   - if a new generated environment was created, read it in full;
   - merge the generated variables into the original active environment via GET -> merge -> PUT, preserving:
     - active flag,
     - environment name,
     - pre-existing variables,
     - generated variable names exactly as created;
   - delete the generated extra environment after successful merge.
10. Final structural verification:
   - confirm the actual generated port/test suites exist under the requested target subtree,
   - confirm the active environment contains the expected merged variables when `local_managed` mode was used,
   - confirm no extra generated environment remains after normalization,
   - confirm the original active environment remains active.

## 6.1) Validated Placement Rule (Required)
- Runtime validation showed nested-parent generation is not reliably represented by the POST response id.
- Example validated behavior:
  - request parent: `.../Test Suite/SOAP`
  - POST response id: root-level named suite id under `.../Test Suite/...`
  - actual generated content: port suite under `.../Test Suite/SOAP/...`
- Therefore:
  - trust follow-up readback under the target subtree,
  - do not drive subsequent mutations from the POST response id alone when the parent is nested.

## 6.2) Validated Environment Behavior (Required)
- Runtime validation showed that using `addNewEnvironment.name=<existing-active-environment-name>` does not append into the active environment.
- Observed behavior:
  - existing active `Local` remained active and unchanged,
  - generation created a new inactive suffixed environment such as `Local 2`,
  - non-prefixed generation produced variables such as `WSDL` and `ENDPOINT` in that extra environment,
  - prefixed generation produced variables such as `PARABANK_WSDL` and `PARABANK_ENDPOINT` in that extra environment.
- Therefore:
  - treat `addNewEnvironment` as a new-environment path,
  - use prefixing when collisions are expected,
  - normalize generated variables into the intended active environment after generation.

## 6.3) Validated Non-Default Reference-Environment Branch
- Runtime validation showed that `referenceExistingEnvironment.file` does work structurally, but does not behave like a safe replacement for `local_managed` normalization into the active local environment.
- Observed behavior:
  - generation added a new inactive referenced environment node such as `Environment Reference 2`,
  - that node pointed at the requested external `.env` file,
  - the pre-existing active local environment remained active,
  - generated SOAP Clients still resolved `${ENDPOINT}` from the active local environment at runtime rather than switching to the referenced file.
- Proof point from runtime validation:
  - after mutating the active local environment to an intentionally wrong endpoint value, the generated `requestLoan` traffic used that wrong active-environment value (`POST /parabank/services/SHOULD_NOT_BE_USED HTTP/1.1`), confirming runtime resolution did not move to the referenced `.env` file.
- Therefore:
  - treat this branch as validated but non-default,
  - use it only when the user explicitly wants a referenced environment node created and understands that runtime still follows the active local environment,
  - do not treat it as equivalent to the safer `local_managed` generation-plus-normalization path.

## 7) Canonical Payloads (API-First)
Disabled environment generation:
```json
{
  "parent": { "id": "/TestAssets/MySuite.tst/Test Suite/SOAP" },
  "name": "ParaBank Generated",
  "location": { "url": "http://localhost:8090/parabank/services/ParaBank?wsdl" },
  "createEnvironment": { "enabled": false }
}
```

`reference_external` generation (validated, non-default):
```json
{
  "parent": { "id": "/TestAssets/MySuite.tst/Test Suite/SOAP" },
  "name": "ParaBank Generated",
  "location": { "url": "http://localhost:8090/parabank/services/ParaBank?wsdl" },
  "createEnvironment": {
    "enabled": true,
    "value": {
      "environmentSource": {
        "referenceExistingEnvironment": {
          "file": {
            "id": "/TestAssets/parabank-wsdl.env"
          }
        }
      },
      "variableTypes": "both"
    }
  }
}
```

`local_managed` generation with prefix:
```json
{
  "parent": { "id": "/TestAssets/MySuite.tst/Test Suite/SOAP" },
  "name": "ParaBank Generated",
  "location": { "url": "http://localhost:8090/parabank/services/ParaBank?wsdl" },
  "createEnvironment": {
    "enabled": true,
    "value": {
      "environmentSource": {
        "addNewEnvironment": {
          "name": "Local",
          "prefix": "PARABANK"
        }
      },
      "variableTypes": "both"
    }
  }
}
```

## 8) Validation
- Expected HTTP status codes:
  - `200` success
  - `400` malformed payload or invalid branch combination
  - `404` invalid parent or unresolved `location`
- Required post-condition checks:
  - actual generated content exists under the intended target subtree
  - active environment remains the intended active environment
  - generated variables are present in the active environment after normalization when `local_managed` mode is used
  - no extra generated environment remains after normalization
  - referenced-environment mode, if used, leaves the expected referenced environment node present and does not silently change the active environment
- Validation boundary:
  - structural/environment readback is required
  - runtime execution is optional and belongs to downstream composite authoring flows after payload values are configured

## 9) Failure Modes
- `404 location.url could not be found` when source URL is not accessible from server runtime.
- `400 Invalid payload` for malformed JSON/schema mismatch.
- Nested-parent placement ambiguity when the POST response id is treated as authoritative without subtree readback.
- API environment divergence from UI expectations:
  - same-name `addNewEnvironment` creates a suffixed extra environment rather than appending to the active one.
- Unsafe non-prefixed generation when collisions already exist in the active environment.
- Incomplete normalization leaving both:
  - the original active environment, and
  - the extra generated environment
  in the same `.tst`.
- Unsafe assumptions about `referenceExistingEnvironment.file`:
  - it creates inactive reference nodes, but runtime still resolves generated-client variables from the active local environment unless that active environment already matches the intended values.

## 10) Safety / Rollback
- Write skill (suite generation plus environment normalization).
- Rollback plan:
  - preserve pre-change snapshots for:
    - target-parent children,
    - environment list,
    - active environment payload;
  - delete generated suite content via `DELETE /v6/suites?id=...` using ids confirmed by post-create readback;
  - restore the original active environment via full read-merge-write payload from the preserved snapshot;
  - delete any generated extra environment that remains.

## 11) Reuse Notes
- Primary target: SOAtest.
- Use Skill 064 as the canonical owner for general environment terminology, external-file mechanics, internal/referenced environment-node lifecycle, and active-environment verification.
- Use this skill when the user wants generated WSDL-based tests added to an existing `.tst`, not a brand-new `.tst`.
- The server API does not expose the UI "Organize as Positive and Negative Unit Tests" toggle for WSDL generation; downstream restructure is the way to achieve that layout when callers need it.
- Use `docs/skills/platform/skill-023-tst-create-from-wsdl.md` for file-level WSDL generation instead.
- Prefer `local_managed` generation plus post-create normalization by default; use the `reference_external` branch only on explicit user intent.
