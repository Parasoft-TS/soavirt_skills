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
- If the operation can be done entirely server-side with a dedicated endpoint, prefer that endpoint.
- Runtime asset targeting policy (mandatory for SOAtest/Virtualize asset tasks):
  - treat server API as the source of truth for resolving runtime targets (`.tst`/`.pva`, suites, tools, data sources, and ids).
  - resolve target identity via API discovery first (for example `GET /v6/children` then descendants/asset reads).
  - do not use local filesystem search (`ripgrep`, glob scans, local path heuristics) to discover runtime server assets.
  - do not inspect repository files, `work/` snapshots, or prior run artifacts as a substitute for API-first runtime asset resolution when the task names a runtime asset.
- API-first mutation policy:
  - if a supported API write path exists, use API mutation.
  - use download/edit/upload only when file content must actually change and no suitable API mutation path is available for the intended operation.
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

### Observed Payload Media-Type Authority (Global)
- For validation-family selection and other payload-sensitive routing, classify response type from the observed semantic payload/body first.
- Treat response headers such as `Content-Type` as supporting evidence only.
- If response headers and the observed payload disagree, follow the observed payload and parser-compatible content shape for tool-family selection.
- Do not attach JSON/XML validation tools only because the producer class or response headers suggest JSON/XML when the observed payload is actually plain text or another type.

### Capability Preflight Gate (Global)
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md` is the canonical preflight policy surface for server-API runtime work.
- Before first write in a branch/session, resolve and normalize the active API base path with a read probe (for example `GET /v6/children`).
- Before first write in a branch/session, run the matched Skill 050 preflight.
- Before first write in a branch/session, this gate is mandatory/fail-closed.
- Prefer verified-supported endpoint-method pairs and documented fallback routes over optimistic assumptions.

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
4. If the task escalates into writes, add the write-branch bundle that matches the mutation class:
  - file writes: platform file ops + capability preflight
  - suite/object writes: structure skill + object/suite PUT safety policy + capability preflight
  - tool writes: target tool skill + tool PUT safety policy + capability preflight
  - output chaining writes: target skill + Skill 017 + Skill 018 + capability preflight
  - execution-observation branches (baseline run, verification run, or traffic observation): apply `Execution Diagnostics Default (Global)`.
This algorithm is mandatory; do not rely on target-card dependency declarations alone.

### Two-Phase Execution Policy for Server Asset Tasks (Required)
Apply the Decision Rule in this sequence for SOAtest/Virtualize runtime asset operations:
1. Phase A - Target resolution (API-first):
  - identify target files/objects/parents via server API reads only.
  - if target cannot be resolved from API, request clarification rather than switching to local filesystem discovery.
2. Phase B - Mutation path selection:
  - use verified-supported API mutation when available.
  - use download/edit/upload only when the selected skill explicitly allows it and Skill 050 fallback routing indicates no suitable API write path for the intended mutation.
This policy prevents local-search misrouting while preserving legitimate YAML fallback workflows.

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
- Optionally review latest transient run artifacts under `work/runs/<date>/<run-name>/` when runtime context is relevant, but treat them as supplementary evidence only; they must not replace API-first target resolution when the active workflow requires server-side discovery.
