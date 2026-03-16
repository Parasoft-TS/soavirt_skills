# Skill 064: SOAtest Environment Lifecycle
## 1) Skill Name
Manage SOAtest environment mechanics across project-scoped external environment files and `.tst`-scoped internal/referenced environments.

## 2) Objective
Provide one canonical v1 owner for environment terminology, mutation, verification, and generation-mode semantics so future agents do not conflate:
- semantic project environments such as `QA` / `DEV`,
- canonical external project `.env` / `.envs` files,
- internal SOAtest Environment objects inside a `.tst`,
- referenced environment nodes inside a `.tst`,
- the `.tst`'s active environment at execution time.

## 3) Scope
- In scope:
  - explain or inspect the environment model for a project or `.tst`
  - create or update project-scoped external `.env` / `.envs` files under `TestAssets/<slug>/environments/`
  - create, read, update, copy, move, and delete internal SOAtest Environment objects through `/v6/environments`
  - create or update caller-owned local environment variables in generated/local environments when downstream broad-authoring branches need normalized values such as `BASEURL`
  - attach referenced external environment files to `.tst` assets
  - verify active-environment state after environment mutation
  - consolidate/fold values from an internal environment into the canonical external project environment file
  - remove redundant internal environments only after successful consolidation and verification
  - define shared generation modes for Skills 022-025:
    - `disabled`
    - `local_managed`
    - `reference_external`
- Out of scope:
  - session-start project bootstrap ownership (owned by Skill 063)
  - broad service-test authoring intake (owned by Skill 033 / Skill 056)
  - assuming runtime switched to an external environment file merely because a referenced environment node exists
  - using referenced environment nodes or external environment files as the default workaround for keeping supported auth secrets out of tracked `.tst` assets during REST Client auth mutation
  - silently inventing environment names, active-environment choices, or merge conflict resolutions

## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-001-shared-introspection.md`
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`
  - `docs/skills/structure/skill-055-testsuite-create-from-wsdl.md`

## 3.2) Ownership Boundaries
- Skill 063 owns:
  - semantic environment capture during project bootstrap,
  - environment-to-service-definition association,
  - canonical external environment-file location under `TestAssets/<slug>/environments/`,
  - durable project-record references to environment files.
- This card owns:
  - external environment-file authoring/update mechanics,
  - internal environment-node lifecycle,
  - referenced-environment attachment mechanics,
  - active-environment verification and post-mutation readback rules,
  - local-to-external consolidation mechanics,
  - shared environment-mode semantics for Skills 022-025
  - generated local-environment variable mechanics for caller-owned post-generation normalization branches such as Skill 065's `BASEURL` insertion.
- `docs/skills/cross-cutting/skill-069-secret-reference-auth-materialization-policy.md` owns the secret-reference-consumption boundary for downstream auth branches; this card must not invent environment indirection to override that policy.

## 4) Canonical Model
Use these layers distinctly:
- `project environment`
  - semantic deployment/test context captured in project context
- `project environment file`
  - canonical external `.env` or `.envs` under `TestAssets/<slug>/environments/`
- `internal environment`
  - local SOAtest Environment object inside a `.tst`
- `referenced environment node`
  - `.tst` environment node that points at an external environment file
- `active environment`
  - execution-time environment actually used by the `.tst`

Canonical mapping:
1. Skill 063 resolves semantic environments and project environment-file references.
2. External environment files hold reusable shared values.
3. `.tst` assets bind to environment data through local/internal or referenced nodes.
4. Runtime follows the verified active environment of the `.tst`.

Conservative v1 runtime rule:
- creating or attaching local/internal or referenced environments is structurally valid but is **not** sufficient evidence that runtime now uses that environment source.
- changing the active environment is a separate explicit action after new environments are added to a `.tst`.
- until a branch verifies active-environment behavior explicitly, treat the `.tst`'s verified active environment as the runtime authority.

## 4.1) Shared Generation Modes
Use these conceptual modes across Skills 022-025:
- `disabled`
  - no environment generation
- `local_managed`
  - for brand-new `.tst` generation, rely on the generator's built-in local/internal environment behavior and then inspect/reuse the generated environment
  - for existing-asset environment branches, generate or maintain local/internal environment data intended to drive runtime
- `reference_external`
  - attach a referenced external environment file

API-shape note:
- OpenAPI and RAML generation use a request field named `createEnvironment`; the value follows the `restCreateEnvironment` schema.
- WSDL and XSD generation also use `createEnvironment`, but with the `createEnvironment` / `variableTypes` schema branch.
- The conceptual modes are shared even though the payload schemas differ.

## 5) Inputs
- Required:
  - resolved project context or explicit project-independent scope
  - target environment action, for example:
    - inspect
    - external-file write/update
    - local/internal environment create/update/delete
    - referenced-environment attachment
    - active-environment verification/change
    - local-to-external consolidation
    - determine generation mode
  - target `.tst` local path and/or environment-node id when `.tst` mutation is in scope
- Optional:
  - semantic project environment name (`QA`, `DEV`, and so on)
  - external environment file path
  - internal environment name
  - merge/override policy when values collide
  - explicit delete-after-consolidation approval
  - generation-mode preference for Skills 022-025

## 6) Preconditions
- If the branch mutates a `.tst` environment object, API reachability/authentication must be confirmed.
- If the branch mutates an external environment file, the project-local path must be resolved and writable.
- If multiple project environments exist and the branch is environment-sensitive, the intended semantic environment must already be resolved or clarified before write.

## 7) Procedure
0. Resolve the branch before mutation:
   - inspect model/state
   - write/update external file
   - create/update/delete local/internal environment
   - attach referenced environment
   - verify/change active environment
   - consolidate local/internal -> external
   - determine generation mode
1. If project-aware context exists, load the active project record first and use it as the source of:
   - semantic environment names,
   - canonical external environment-file path,
   - known service-definition/environment associations,
   - project-wide environment-specific facts.
2. Do not silently override explicit user path/project/environment choices with stored defaults.

### 7.1) External Environment-File Rules
3. Canonical file location:
   - one environment -> `TestAssets/<slug>/environments/<environment>.env`
   - multiple environments -> `TestAssets/<slug>/environments/<slug>.envs`
4. Preserve:
   - existing namespace,
   - existing environment names,
   - unchanged variables,
   - existing variable ordering when the branch does not explicitly require reordering.
5. When service definitions are known, prefer confirmed project-record mappings for `BASE_URL` and related environment-specific values over re-asking or re-deriving them.
6. Verify after write:
   - file exists,
   - parse succeeds,
   - expected environment entries/variables are present,
   - unchanged content was not accidentally removed.

### 7.2) Internal / Referenced Environment Lifecycle
7. For `.tst` environment-node branches, use `/v6/environments` as the API authority for internal/referenced environment identity and state.
8. Create/update branches:
   - local/internal environment -> use the `local` payload branch
   - referenced environment -> use the `reference` payload branch with external file location
9. Required readback after any `.tst` environment mutation:
   - intended node exists,
   - node type is correct (`local` vs `reference`),
   - referenced location is correct when applicable,
   - active flag / active-environment state is read back explicitly rather than assumed.
Boundary note:
- referenced-environment attachment is for environment mechanics, not the default auth-concealment strategy for generated/client auth writes
- use this card for env-driven auth indirection only when the user explicitly asks for environment-based runtime handling

### 7.3) Active-Environment Rule
10. Treat active-environment state as a first-class verification target.
11. If the branch depends on runtime switching:
   - confirm which environment is active after mutation,
   - do not assume active state changed because a reference node was created or because an external file contains a matching environment name.
12. If a runtime-sensitive branch still cannot prove active-environment behavior from supported readback:
   - stop and report the branch as structurally updated but runtime-unverified,
   - do not silently represent that branch as a confirmed runtime switch.

### 7.4) Local-to-External Consolidation Branch
13. For requests such as “fold the internal environment into the external project environment and make the reference active”:
   - inspect the current internal environment contents,
   - inspect the target external environment file and target semantic environment entry,
   - merge values into the external file using explicit override rules:
     - explicit user direction wins,
     - otherwise preserve unchanged external values and add missing internal values,
     - do not silently drop colliding values without reporting the resolution policy.
14. After the external file is updated, attach or update the referenced environment node in the `.tst`.
15. Verify:
   - external file contents,
   - referenced node presence and target file location,
   - active-environment state.
16. Delete the redundant internal environment only when:
   - external-file write succeeded,
   - reference attachment succeeded,
   - active-environment verification succeeded for the branch's required level of confidence,
   - the user explicitly asked for or approved deleting the redundant local environment.
17. If runtime activation remains unverified, keep the internal environment unless the user explicitly accepts a structural-only result.

### 7.5) Generation-Mode Handshake for Skills 022-025
18. When a generation card needs environment strategy:
   - prefer explicit user choice,
   - otherwise use project context and this default rule:
     - `local_managed` is the safe v1 default,
     - `reference_external` is supported but behavior-sensitive,
     - `disabled` is valid only when the user explicitly wants no environment generation or the calling branch requires it.
19. For Skills 022-025 creating a brand-new `.tst`, `local_managed` means the generator-owned local-environment path:
   - rely on the service-definition generator's built-in local environment behavior,
   - inspect/read back the generated environment after create and reuse it as the initial environment context,
   - do not perform a separate `/v6/environments` create/update as part of initial generation unless the user explicitly requested a follow-on environment task.
20. Treat `createEnvironment` / `restCreateEnvironment` as configuration surfaces, not agent-discretion surfaces:
   - include them only when the current request, or already-confirmed upstream context for this exact branch, explicitly resolves a non-default environment path,
   - examples include `disabled`, `reference_external`, or a local-environment branch requiring concrete request fields such as `prefix` or WSDL/XSD `variableTypes`.
21. Generation cards should reference this skill for environment-mode semantics rather than redefining those modes locally.
22. When a caller immediately follows brand-new generation with template normalization, this card owns the generated local-environment variable mechanics only:
   - inspect the generated local environment
   - create or update caller-requested variables such as `BASEURL` using confirmed project-context or service-definition values
   - verify the variable via environment readback before downstream client rewrites continue
   - leave generated REST Client URL rewriting to the caller and Skill 020 rather than absorbing that mutation into environment ownership
23. Evidence-backed OpenAPI rule for `reference_external` generation:
   - the request field name remains `createEnvironment`; the value shape is `restCreateEnvironment`
   - attached external environment files are not consulted to resolve `location.url=${...}` during the create request itself; probes with `${OPENAPI}` and `${PARABANK_SWAGGER}` in `location.url` failed before generation
   - when the create call instead uses a concrete OpenAPI URL and the attached external environment file contains matching OpenAPI/base-url variables, the generator may emit those external variable names into the generated `.tst`
   - observed emitted-name cases in this workspace include `${OPENAPI}` / `${BASEURL}` and `${PARABANK_SWAGGER}` / `${PARABANK_BASEURL}`
   - attached external env files that did not present a generator-recognized matching base-url variable still produced concrete generated REST Client URLs in this workspace
   - therefore callers must inspect both the external environment contents and the generated `.tst` readback before deciding whether further local-variable insertion or REST Client URL rewrite is still required

## 8) Validation
- External environment-file writes are verified by file existence + parseability + expected content presence.
- Internal environment writes are verified by `/v6/environments` readback.
- Caller-owned post-generation local-variable insertion (for example `BASEURL`) is verified by readback of the generated local environment before downstream client normalization continues.
- Referenced-environment attachment is verified by readback of the reference node and target file location.
- Active-environment-dependent branches never infer runtime authority from structure alone.
- Local-to-external consolidation never deletes the local/internal source environment before consolidation and reference verification succeed.

## 9) Failure Modes
- Semantic project environments, external environment files, internal environments, referenced nodes, and active environments are treated as synonyms.
- A referenced environment node is created and the agent incorrectly reports that runtime must now be using the external file.
- External-file updates overwrite unrelated environment entries or variables.
- Internal-environment values are folded into the wrong semantic environment entry in the external file.
- Redundant internal environments are deleted before reference attachment and active-environment verification succeed.
- Environment mechanics are misused as the default workaround for keeping supported auth secrets out of a generated `.tst`.
- A caller rewrites generated REST Clients to `${BASEURL}` without first inserting and readback-confirming the local `BASEURL` variable in the generated environment.
- Generation cards assume `reference_external` is runtime-equivalent to `local_managed` without verification.
- OpenAPI generation branches assume `BASE_URL` versus `BASEURL` alone determines parameterization without inspecting the generated `.tst` readback.
- Callers assume generic `${OPENAPI}` / `${BASEURL}` names are always the correct replacements instead of reading the attached external environment file and preserving any already-generated prefixed names.

## 10) Safety / Rollback
- Fail closed on ambiguous environment selection when the branch is environment-sensitive.
- Keep rollback material before any destructive local/internal environment deletion:
  - original external environment file content,
  - original `/v6/environments` readback for affected nodes,
  - explicit record of active-environment state before mutation.
- For local file edits, use Skill 006 when rollback-preserving local YAML/file handling is required.

## 11) Reuse Notes
- Use this skill whenever a task needs environment mechanics, even if the broader workflow was reached through another orchestration card.
- Skill 063 should remain the semantic/bootstrap owner and should reference this card for external environment-file mechanics rather than embedding the full XML/mutation procedure itself.
- Skill 055 remains an important validated evidence source for WSDL-specific behavior, but generalized environment rules should be read from this card first.
- Skill 065 may use this card immediately after Skill 022 generation to add a local `BASEURL` variable before the caller normalizes generated template-suite REST Client URLs.
