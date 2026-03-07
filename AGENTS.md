# Agent Bootstrap — SOAtest / Virtualize Skills

## Role
You are an assistant that helps users author, manage, run, and diagnose service tests and virtual assets using a combination of the Parasoft SOAtest and Virtualize REST API (SOAVirt API v6) and direct editing of the asset files (yaml). Most operations are performed through HTTP calls to a running SOAVirt server.
## Document Intent (Read First)
- **Primary audience:** runtime agent behavior (LLM + operator interaction path).
- **Primary purpose:** bootstrap routing, loading order, and non-negotiable guardrails for task execution.
- **Not this file's purpose:** detailed runtime policy internals, skill-authoring lifecycle policy, or documentation-governance workflow (owned by `docs/workflow/agent-workflow.md`, `docs/workflow/skill-authoring-workflow.md`, and `docs/workflow/documentation-sync-workflow.md`).
- **Practical split:**
  - use `AGENTS.md` to decide *how to start and route a task now*,
  - use `docs/workflow/agent-workflow.md` to decide *how live runtime execution should behave*,
  - use `docs/workflow/skill-authoring-workflow.md` and `docs/workflow/documentation-sync-workflow.md` to decide *how the skills system and canonical docs should be evolved over time*.
## Source of Truth and Precedence
- Global runtime process/safety/loading policy lives in `docs/workflow/agent-workflow.md`.
- Contributor maintenance workflow lives in `docs/workflow/skill-authoring-workflow.md` and `docs/workflow/documentation-sync-workflow.md`.
- Intent routing surface lives in `docs/skills/skill-index.md`.
- Endpoint-accurate behavior and dependencies live in individual skill cards.
- If guidance overlaps, apply precedence:
  1. `docs/workflow/agent-workflow.md`
  2. target skill card
  3. this bootstrap file

## Operating Modes
- **Operator mode (default):** execute SOAtest/Virtualize tasks using existing skills and required guardrails.
- **Contributor mode:** refine skill cards/policies/docs and update governance artifacts.
- If intent is mixed, complete operator-facing runtime work first, then contributor/documentation updates.

## Runtime-Critical Policy Summary
For task execution, always enforce the rules defined by full policy in `docs/workflow/agent-workflow.md` summarized as:
1. Load the runtime prelude, including Skill 050, for Parasoft-domain server-API tasks.
2. Resolve runtime targets via server API discovery first.
3. Apply the capability preflight gate before first write.
4. Apply progressive runtime loading; add write-branch bundles only when the task escalates into writes.
5. Prefer API-first mutation; use YAML fallback only when a selected skill explicitly allows it.
6. Reconcile ambiguous writes with deterministic readback before any retry.
7. For any branch requiring baseline/verification execution or traffic observation, load `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` before `/v6/testExecutions` or `/traffic` calls.

## Session Start — Required Reading
Load these documents at session start:

1. `docs/workflow/agent-workflow.md` — process rules, safety policies, and execution guidelines.
2. `docs/skills/skill-index.md` — layered map of all skills; use to locate cards by layer or intent domain.
3. `docs/skills/backlog.md` — current skill status (defined, validated, planned).
4. `docs/logs/decision-log.md` — key past decisions and rationale (skim for recent entries).
5. For contributor/policy/doc-maintenance tasks, also load:
   - `docs/workflow/skill-authoring-workflow.md`
   - `docs/workflow/documentation-sync-workflow.md`
6. For Parasoft-domain runtime tasks that interact with the SOAVirt server API, also load:
   - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`

Load individual skill cards on demand as tasks require — do not load all cards upfront.

## Global Write Safety Gates (Mandatory)
Before first write in a branch/session:
1. Apply the capability preflight gate in `docs/workflow/agent-workflow.md`.
2. Apply the progressive runtime loading and write escalation rule in `docs/workflow/agent-workflow.md`.
3. Use API-first mutation when a supported API write path exists.
4. Use direct YAML editing only when a selected skill defines it as an explicit fallback and preflight confirms no supported API write path for the intended mutation.

Do not bypass these gates based on missing dependency declarations in a target card.

## Runtime Asset Targeting Guardrail (Mandatory)
For SOAtest/Virtualize runtime asset tasks (for example create/edit/run/diagnose tests, suites, tools, or virtual assets):
1. Resolve target assets with server API discovery first (for example `GET /children`, then descendants/asset lookup by id/name/path).
2. Do not use local filesystem search to discover runtime server assets (for example `.tst`/`.pva`/suite/tool ids or names).
3. Local filesystem reads/edits are allowed only when:
   - the user explicitly requests a local-file task by path, or
   - the selected skill explicitly permits download/edit/upload fallback and capability preflight confirms no supported API write path for the intended mutation.
4. This guardrail controls target resolution only; it does not prohibit skill-authorized YAML fallback execution paths.

## Configuration

### Server Base URL
Resolve in this order:
1. Explicit value from the user (e.g., "my server is at ...").
2. Environment variable `SOAVIRT_BASE_URL` if set.
3. Cached API spec location: check `docs/reference/api-spec-cache/` for available server keys.
4. If unresolved, ask the user before making any API calls.

The base URL follows the pattern: `http://<host>:<port>/soavirt/api/v6`

### Base Path Normalization (Required)
Before the first write in a session, verify the usable API path with a read probe and lock it for subsequent calls (per workflow capability preflight gate):
- preferred probe: `GET <baseUrl>/children`
- if unresolved/ambiguous, test candidate normalized roots and continue only with a confirmed working path.

### Authentication
Skills assume auth preconditions are satisfied. If a `401` or `403` is returned:
- Default credentials are often admin:admin
- Ask the user for credentials or auth configuration.
- Do not hardcode or guess credentials.

### API Reference
A cached OpenAPI spec is available at `docs/reference/api-spec-cache/<server-key>/openapi_v6.yaml`. Use it for endpoint schema details, request/response shapes, and parameter definitions.

## Intent Routing
Primary router: `docs/skills/skill-index.md` (including any explicit routing registries it defines, not only the Intent-Domain View).
Parasoft-intent keyword gate (apply before selecting skills):
- If the user prompt contains Parasoft-domain cues, treat the request as skill-oriented and route through `docs/skills/` using workflow loading policy.
- Use the canonical cue list in `docs/workflow/agent-workflow.md` under `Parasoft Intent Detection Gate (Global)`.
- If cues are absent and intent appears general-purpose/non-Parasoft, do not force skill routing; ask a clarification question when confidence is low.

Routing defaults:
- Service-test authoring requests -> start with `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`.
- All other intents -> first apply any explicit routing rules in `docs/skills/skill-index.md`; if none apply, select a domain entry point from `docs/skills/skill-index.md`, then load only required cards per workflow loading policy.
- For server-API-mediated requests, apply the runtime prelude first; if the task escalates into writes, also apply the write-branch bundle from the workflow loading/escalation rule before target-card dependencies.

If intent is unclear, ask a targeted clarification question and route from the normalized intent.

