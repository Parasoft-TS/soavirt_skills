# Agent Runtime Workflow
## Purpose
Define the global runtime execution policy for SOAtest and Virtualize tasks handled through this skills project.
## Document Scope & Audience
- **Primary audience:** runtime agents and operators executing live SOAtest/Virtualize tasks.
- **Primary purpose:** global policy surface for runtime loading, safety, routing, evidence boundaries, and failure handling.
- **Relationship to `AGENTS.md`:**
  - `AGENTS.md` is runtime bootstrap/routing guidance,
  - this workflow doc owns the canonical runtime policy that `AGENTS.md` summarizes.
- **Not this document's purpose:**
  - skill-authoring and long-term maintenance workflow (owned by `docs/workflow/skill-authoring-workflow.md`)
  - canonical-doc/log synchronization workflow (owned by `docs/workflow/documentation-sync-workflow.md`)
- **Usage rule:** if guidance overlaps, keep high-level bootstrap in `AGENTS.md`, keep runtime-global policy here, and keep contributor-maintenance policy in the companion workflow docs.

### Runtime Prelude (Global)
- For any Parasoft-domain runtime task that interacts with the SOAVirt server API, load:
  - `docs/workflow/agent-workflow.md`
  - `docs/skills/skill-index.md`
  - `docs/skills/backlog.md`
  - `docs/logs/decision-log.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
### Incremental Skill Loading Rule (Global)
- After the runtime prelude, load only the minimum extra cards for a task: one target card + direct dependencies.
- When a skill lists cross-cutting dependencies, load them before executing the skill's procedure. If two skills address the same domain (for example JSON validation tools), load ALL listed dependencies from BOTH skills before proceeding.
- Skill-declared dependencies are additive safeguards, not the only safeguards.
- Do not allow missing dependency declarations in a target card to bypass mandatory prelude safeguards.
### Session-Start Mode Classification Rule (Global)
- After the session-start required reading defined in `AGENTS.md`, classify the user's bootstrap prompt before auto-entering Skill 063.
- Treat explicit contributor-start cues (for example `Follow @AGENTS.md for skill contributor workflow`, `Follow @AGENTS.md for skill authoring workflow`, or other equally explicit skill-maintenance starts) as contributor bootstrap: load contributor-maintenance workflow surfaces and do not auto-run Skill 063 unless the user also asks for project context or later shifts into project-aware runtime work.
- Treat explicit project/operator-start cues (for example naming a project, asking to load project context, or otherwise requesting project-aware SOAtest/Virtualize runtime work) as operator bootstrap.
- Treat bare or under-specified `Follow @AGENTS.md`-style startup prompts as the operator default.
- If explicit contributor and project/runtime cues conflict in the same startup prompt, ask one targeted clarification question before continuing.
### Session-Start Project Bootstrap Rule (Global)
- When session-start classification resolves to operator bootstrap, immediately after the session-start required reading defined in `AGENTS.md`, load `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md` and run project bootstrap before any further project-aware task work.
- This automatic entry includes bare or under-specified `Follow @AGENTS.md`-style starts and explicit project/operator starts.
- Do not auto-run Skill 063 for explicit contributor-start prompts unless the user later asks to load/store/update project context or shifts into project-aware runtime work.
- At a high level, bootstrap must either:
  - resolve one existing project and continue with that project's durable context
  - ask one disambiguation question when several close matches exist
  - create, update, or repair a durable project record under `TestAssets/` when needed
- Preserve progressive context loading after bootstrap:
  - resolve project context in this order: `TestAssets/project-index.yaml` -> selected `TestAssets/<slug>/<slug>.yaml` -> referenced environment file(s) under `TestAssets/<slug>/environments/` -> only the specific referenced files needed for the active task
  - do not preload all project references, notes, or history by default
- Skill 063 owns the exact prompts, matching thresholds, persistence steps, update flow, and summary contract.
### Post-Bootstrap Project Context Contract (Global)
- Once Skill 063 resolves an active project, that project record becomes part of session runtime state for all later project-aware work until the user changes projects or the branch is clearly project-agnostic.
- Direct routing from `docs/skills/skill-index.md` to a smaller atomic/workflow card, or handoff from one card to another, does not clear that active-project obligation.
- For project-aware branches, consult the active project record before:
  - asking the user for information that may already be stored,
  - choosing default file-backed asset locations,
  - synthesizing request/configuration values,
  - resolving schema/service-definition sources when the owning workflow still needs source selection,
  - making validation-priority or environment-sensitive decisions.
- Shared consultation order for project-aware branches:
  1. explicit current-task instructions,
  2. active project record sections relevant to the task (`environment_files`, `services`, `facts`, `references`, `notes`),
  3. values already confirmed in the current session,
  4. direct runtime evidence or contract evidence appropriate to the current branch,
  5. same-`.tst` local evidence when the owning skill allows it,
  6. one targeted user question.
- Relevance gate:
  - project-record consultation is mandatory only when the branch still involves asset placement/default destinations, request/config synthesis, schema/service-definition source resolution, validation priorities/business rules, or environment-specific behavior.
  - if those decisions are already fully specified, the smaller card may proceed without mechanically reloading all project context while still treating the active project as in scope.
- Override rule:
  - explicit current-task instructions outrank stored project defaults for target paths, project selection, environment choice, source location, and destination location.
  - active project context supplies defaults, durable references, and missing facts; it must not silently override explicit user scope.
  - intentional cross-project work is valid and must not be collapsed back into the currently active project.
### Secret Reference Consumption and Auth Materialization Boundary (Global)
- Skill 063 owns durable secret-reference storage and project-record path references; that storage rule does not prohibit later runtime consumption.
- When a runtime branch needs credentials and an approved gitignored secret reference exists in project context, the owning workflow/leaf may resolve those values for direct exploration, execution, or supported `.tst` auth mutation.
- For supported auth-write surfaces, materializing the resolved values into the `.tst` through the owning leaf is valid and is not by itself a `blocked-auth` condition.
- Do not detour into referenced-environment or other concealment workarounds solely to avoid credential persistence unless the user explicitly asks for that strategy.
- Protecting generated assets before source-control commit remains the user's responsibility unless an owning workflow explicitly states otherwise.
### SOAtest Environment Model and Owner (Global)
- Use these terms distinctly:
  - `project environment`: semantic deployment/test context captured during bootstrap (for example `QA`, `DEV`)
  - `project environment file`: canonical external `.env` / `.envs` stored under `TestAssets/<slug>/environments/`
  - `internal SOAtest Environment object`: environment node inside a `.tst`, managed through `/v6/environments`
  - `referenced environment node`: `.tst` environment node that points at an external environment file
  - `active environment`: execution-time environment actually used by a `.tst`
- Canonical mapping:
  - Skill 063 owns semantic environment capture and durable project environment-file references,
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md` owns environment mechanics across external files, internal nodes, referenced nodes, active-environment verification, and shared generation-mode semantics,
  - runtime behavior follows the verified active environment of the `.tst`, not merely the existence of a referenced environment node or project environment file.
- Runtime rule:
  - creating or attaching local/internal or referenced environments does not by itself prove that execution has switched to that environment source.
  - changing the active environment is a separate explicit action after new environments are added to a `.tst`.
  - until a branch verifies active-environment behavior explicitly, treat the `.tst`'s verified active environment as the runtime authority.
### Parasoft Intent Detection Gate (Global)
- Before selecting cards, detect whether the user request is Parasoft-domain or general-purpose.
- Treat requests as Parasoft-domain when prompt cues include (case-insensitive, singular/plural allowed):
  - `tst`, `api test`, `api testing`, `soatest`, `pva`, `virtual service`, `service mock`, `api mock`, `parasoft`, `service virtualization`
- For Parasoft-domain requests:
  - route via `docs/skills/skill-index.md`,
  - apply any explicit routing registries/tie-break rules defined in `docs/skills/skill-index.md` before falling back to layer/domain navigation,
  - apply progressive loading and safety rules defined in this workflow.
- For clearly non-Parasoft requests:
  - do not force skill loading from `docs/skills/`.
- When confidence is low or cues conflict, ask one targeted clarification question before mutation.

### Per-Turn Intent Re-Routing Rule (Global)
- On every new user prompt, re-run intent routing from `docs/skills/skill-index.md` before reusing the previously selected target skill.
- Do not assume continuity of prior skill selection when user intent shifts (for example deep analysis -> high-level summary, or summary -> datasource introspection).
- If a new prompt maps to a different routing rule/card, switch to that target card and load only required dependencies for the new branch.
- If intent remains ambiguous after re-routing, ask one targeted clarification question before proceeding.

### Read-Only Analysis Routing Rule (Global)
- For Parasoft-domain read-only analysis requests, consult `docs/skills/skill-index.md` routing rules before selecting a skill card.
- When a routing rule in `docs/skills/skill-index.md` selects an analysis card, treat that card's evidence policy and output contract as binding.
- If a read-only analysis request remains ambiguous after applying the routing rules, prefer the more evidence-constrained matching analysis skill unless the user explicitly asks for a higher-level summary.

### Decision Rule
- If the operation is a pure local file-backed asset operation, prefer the local workspace path and do not force API discovery first.
- If the operation can be done entirely server-side with a dedicated endpoint, prefer that endpoint.
- File-backed asset targeting policy (mandatory for file-backed asset tasks):
  - treat explicit local paths and merged-workspace search as the source of truth for locating file-backed assets such as `.tst`, `.pva`, `.pvn`, env files, and project resources.
  - use active-project-local search and likely-root-first search before broadening to other local roots/projects.
  - do not force API discovery first when a file-backed asset has already been resolved locally.
- Runtime object targeting policy (mandatory for runtime-object tasks):
  - treat the localhost API as the source of truth for runtime object ids (`testSuite`, tool, datasource, environment, output-provider, execution target, and similar in-file runtime identities).
  - resolve runtime object identity via API discovery when the task depends on those ids.
  - do not infer runtime object ids from local files alone.
- API-first mutation policy:
  - if a supported API write path exists, use the smallest validated API-backed owner for that mutation class.
  - ordinary rename, copy/move/delete, enable/disable, and routine configuration updates must not fall back to local YAML merely because a narrow owner has not been loaded yet.
  - use local YAML fallback only when file content must actually change, the selected skill explicitly allows local YAML editing, and no suitable validated API mutation path exists for the intended operation.
- For assertion authoring, compose logic from the user prompt at runtime (field/rule/source), and avoid codifying domain-specific fixed assertion recipes as mandatory patterns.
- For Diff/Assertor stabilization branches, summarize runtime differences in human-readable form (field/XPath/property and expected vs actual) and require explicit user confirmation before adding ignore rules.
- Build new tool configurations from endpoint contracts and skill rules first; existing workspace tool instances may be used only as optional verification, never as a required source template.
- Exemplar-lookup efficiency guard:
  - when the selected skill already provides sufficient canonical payload-shape guidance, do not scan the workspace for same-type tool exemplars before create/update.
  - prefer user inputs + endpoint schema + skill guidance; use workspace exemplars only as optional troubleshooting evidence after a concrete payload-shape issue appears.

### Approved-Plan Preservation Rule (Global)
- When a composite/orchestration card presents a per-target execution plan and the user explicitly approves it, downstream leaf skills must treat that plan as binding for that branch.
- After approval, leaf skills may refine only configuration details within the approved family and intent.
- Leaf skills must not silently:
  - change tool family,
  - omit approved coverage,
  - downgrade schema validation to content-only validation,
  - convert approved field/assertor intent into full-body diffing,
  - add ignored-difference rules.
- If an approved tool cannot be created or configured, mark that target `blocked` or `partial`, report the reason, and return for explicit user decision rather than substituting another family automatically.

### Mandatory-Phase Integrity Rule (Global)
- When the selected workflow or skill defines in-scope phases as required or mandatory, treat those phases as part of the delivered scope regardless of whether the run is interactive, autonomous, or delegated through another card.
- Required phases remain required unless:
  - the user explicitly narrows scope, or
  - the canonical card marks the phase `Optional / Deferred`, not applicable, or otherwise outside the selected branch.
- Do not reinterpret early success, partial runtime success, task size, local uncertainty, representative coverage elsewhere, or "later phase not reached yet" as permission to skip a later required phase, to report the workflow as effectively complete, or to emit a terminal blocker state while a next deterministic in-scope action still exists.
- Distinguish downstream required work that is still pending behind unfinished upstream authoring from a true terminal blocker:
  - phase-pending work is still in scope and must remain visible, but it is not by itself a reason to stop,
  - if some slices or targets are blocked while others still have a deterministic safe continuation path, keep the actionable slices moving and aggregate the unresolved blockers rather than stopping the whole workflow immediately,
  - reserve terminal `partial`/`blocked` outcomes for cases where reasonable continuation has been attempted and no next safe in-scope action remains.
- If a later required phase cannot be completed credibly after reasonable continuation, stop as `partial` or `blocked` and report the exact unfinished phase and reason rather than silently narrowing the workflow.

### Observed Payload Media-Type Authority (Global)
- For validation-family selection and other payload-sensitive routing, classify response type from the observed semantic payload/body first.
- Treat response headers such as `Content-Type` as supporting evidence only.
- If response headers and the observed payload disagree, follow the observed payload and parser-compatible content shape for tool-family selection.
- Do not attach JSON/XML validation tools only because the producer class or response headers suggest JSON/XML when the observed payload is actually plain text or another type.

### Experimental Exploration Lane Boundary (Global)
- Direct endpoint exploration evidence is a first-class baseline only inside the explicit opt-in experimental lane owned by `docs/skills/composite-orchestration/skill-065-experimental-live-exploration-service-test-orchestration.md`, `docs/skills/cross-cutting/skill-066-experimental-direct-api-exploration-evidence-policy.md`, and `docs/skills/composite-orchestration/skill-067-experimental-validation-enrichment-orchestration.md`.
- Outside that explicit lane, keep the stable execution-backed evidence model owned by Skills 033, 057, and 058.
- Do not generalize the experimental exploration baseline into stable routing or stable validation/remediation branches without an explicit later architectural change.

### Capability Preflight Gate (Global)
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` is the canonical preflight policy surface for API-based runtime work.
- Use the lightweight Skill 050 lane for API-read branches that need safe base-path, endpoint-method, and `404` handling.
- Before first API mutation or execution branch in a session, resolve and normalize the active API base path with a read probe (for example `GET /v6/children`) and run the matched Skill 050 preflight.
- The write/execution lane is mandatory/fail-closed before continuation.
- Prefer verified-supported endpoint-method pairs and documented fallback routes over optimistic assumptions.

### SOAVirt Base URL Bootstrap Rule (Global)
- Before asking the user for `SOAVIRT_BASE_URL` or a server base URL, inspect repo-local SOAVirt reference material first:
  - `docs/reference/api-spec-cache/README.md`
  - `docs/reference/api-spec-cache/<server-key>/openapi_v6.yaml`
- If exactly one usable cached server key or other repo-local SOAVirt reference yields a single plausible base URL candidate, derive `http://<host>:<port>/soavirt/api/v6` from that local evidence and use it for the required read probe.
- Ask the user for base-URL help only when no usable local reference exists, multiple plausible candidates remain, or all locally derived candidates fail confirmation.
- This bootstrap rule is only for resolving the SOAVirt server API base URL; it does not authorize local-file discovery of runtime assets that still require API-first target resolution.

### Payload Encoding Guardrails (Global)
- For any JSON write request (`POST`/`PUT` with `application/json`), require UTF-8 **without BOM** payload bytes.
  - Avoid PowerShell 5.1 patterns that emit BOM for request files.
  - Prefer in-memory JSON body generation or explicit no-BOM writes before `--data-binary @file`.

### Output Chaining Authority (Global)
- For output-chaining writes, use `docs/skills/cross-cutting/skill-017-output-chaining-model.md` + `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` as the canonical policy surface.
- Treat Skill 018 as fail-closed mapping authority for producer/output-provider parent resolution.

### Execution Diagnostics Default (Global)
- For baseline runs, verification runs, and traffic-observation branches, load `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` before any `/v6/testExecutions` or `/traffic` calls.
- Treat Skill 012 payload schema as canonical for execution payload construction.
- Use the execution triad from `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` and correlation guidance in `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`.

### Progressive Runtime Loading and Write Escalation (Required)
For Parasoft-domain requests, load in this order:
1. Runtime Prelude (above).
2. Route via `docs/skills/skill-index.md` and load the minimal target card for user intent.
3. Load target-card declared dependencies (additive).
4. If the task remains local file-backed work, stay in the local-first branch and load only the local asset/path surfaces needed for that work.
5. If the task escalates into an API branch, enter Skill 050 with the matching branch profile before continuing.
6. If the task escalates into writes or execution, add the branch bundle that matches the mutation class:
  - local file writes: platform file ops / local path handling, plus Skill 006 only when local YAML editing is explicitly authorized by the owning leaf
  - suite/object writes: choose the smallest validated owner (for example suite lifecycle, rename-object, or disabled-state mutation) + object/suite PUT safety policy where applicable + capability preflight
  - tool writes: choose the smallest validated owner (for example target tool lifecycle/config card, rename-object, generic tool copy/move/delete leaves, or disabled-state mutation) + tool PUT safety policy where applicable + capability preflight
  - output chaining writes: target skill + Skill 017 + Skill 018 + capability preflight
  - execution-observation branches (baseline run, verification run, or traffic observation): apply `Execution Diagnostics Default (Global)`.
This algorithm is mandatory; do not rely on target-card dependency declarations alone.
### Two-Lane Execution Policy for Asset Tasks (Required)
Apply the Decision Rule in this sequence for SOAtest/Virtualize asset operations:
1. Phase A - Target resolution:
  - for file-backed assets, resolve the target by explicit local path or merged-workspace search first
  - for runtime objects, resolve the target id through the localhost API
  - if the correct lane still leaves the target ambiguous or unresolved, request clarification rather than guessing
2. Phase B - Operation path selection:
  - use local filesystem operations for pure file-level reads/copy/rename/delete work
  - use verified-supported API mutation or execution paths when the branch is semantic, generative, or execution/diagnostic in nature
  - use local YAML fallback only when the selected skill explicitly allows it and the local rollback/preflight requirements for that branch are satisfied
This policy prevents API-first misrouting for file-backed assets while preserving API authority where runtime semantics still require it.

### Failure Response Rule (Global)
- `401`/`403`: stop writes and resolve credentials/licensing.
- `404`: re-resolve identifiers or active base path, then re-enter Skill 050 `404` disambiguation when needed.
- `405`: stop retry loops and reroute using Skill 050 fallback selection.
- `400`: correct payload shape, schema, or encoding before retry.

### Ambiguous Write Recovery Policy (Global)
- For any write call (`POST`/`PUT`/`PATCH`/`DELETE`), if command output is truncated, missing, or otherwise ambiguous, do not retry immediately.
- First run deterministic readback reconciliation for the same target and intended effect.
- Retry is allowed only when reconciliation confirms the intended write did not occur.
- For create flows, treat "artifact found by exact id/name after ambiguous write" as success and continue downstream steps.
- Before any create retry, run a same-name collision read check and branch to safe handling (reuse/delete-and-recreate/new-name) when the artifact already exists.

### Multi-Step Orchestration Ownership (Global)
- Use `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` as the canonical interaction model for underspecified service-test-authoring intake, conversational scope selection, plan confirmation, approved-plan continuation, and staged execution where ambiguity still requires orchestration.
- Keep this workflow doc focused on global safety/loading/failure policy.
- Do not duplicate full conversational intake scripts or branch-specific orchestration logic in this file when Skill 033 already owns it.
- Other workflow/composite cards may define branch-specific deltas or repeated clarified workflows relative to Skill 033, including direct validation-enrichment workflows once authoring scope is already clear; `docs/skills/skill-index.md` owns when direct routing should bypass Skill 033 and target those cards instead.

## Session Continuity
- After session compaction or continuity loss, re-intake the canonical runtime surfaces before proceeding:
  - `docs/workflow/agent-workflow.md`
  - `docs/skills/skill-index.md`
  - `docs/skills/backlog.md`
  - `docs/logs/decision-log.md`
  - include `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` for server-API runtime work
- Use `docs/skills/skill-index.md` as the routing surface after re-intake.
- If the prior session under this repo had an active project, re-load the selected `TestAssets/<slug>/<slug>.yaml` before task-specific execution and then load only the referenced environment/reference files needed for the resumed task.
- Treat the previously active project as still in scope after re-intake for project-aware branches unless the user explicitly changes projects or the resumed task is clearly project-agnostic.
- If the user later names a different project, supplies an explicit path rooted in another project, or otherwise changes project scope mid-session, update active-project state through Skill 063 before continuing project-aware work.
- If continuity resumes with only a partial project record loaded, reload the needed portions of the active project context instead of assuming either that all project context is known or that it has been lost entirely.
- After `AGENTS.md`/startup re-intake in a resumed session under this repo, re-apply the session-start mode classification rule: operator/default resumes that still need project-aware runtime work should reload project context, while explicit contributor-start resumes do not auto-enter Skill 063 unless project context is requested.
- Optionally review latest transient run artifacts under `work/runs/<date>/<run-name>/` when runtime context is relevant, but treat them as supplementary evidence only; they must not replace API-first target resolution when the active workflow requires server-side discovery.
