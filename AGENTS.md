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
Use `docs/workflow/agent-workflow.md` as the canonical owner of runtime-global policy. Before acting, make sure you:
1. load the runtime prelude, including Skill 050 for server-API work
2. route from `docs/skills/skill-index.md`
3. resolve runtime targets through server API discovery before choosing a mutation path
4. run capability preflight before first write and apply progressive write-branch loading
5. use Skill 012 before execution/traffic branches and preserve any user-approved orchestration plan through downstream execution

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
Before first write, apply the capability preflight gate and progressive runtime loading rules from `docs/workflow/agent-workflow.md`. Use API-first mutation unless a selected skill explicitly permits YAML fallback after preflight confirms that no supported API write path exists for the intended change.

## Runtime Asset Targeting Guardrail (Mandatory)
Resolve runtime assets through server API discovery first. Do not substitute repository files, `work/` artifacts, or local filesystem search for runtime target resolution. Local file reads/edits are allowed only for explicit local-file tasks or skill-authorized download/edit/upload fallback branches after preflight.

## Configuration

### Server Base URL
Resolve in this order:
1. explicit value from the user
2. environment variable `SOAVIRT_BASE_URL`
3. one unambiguous repo-local SOAVirt cache/reference candidate from `docs/reference/api-spec-cache/`
4. otherwise ask the user after local candidates remain unresolved, ambiguous, or fail read-probe confirmation

Do not ask for a base URL merely because the environment variable is unset when the repo already provides one unambiguous local candidate. Use the `http://<host>:<port>/soavirt/api/v6` form and follow the workflow path-normalization rule before writes.

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
Primary router: `docs/skills/skill-index.md`.
- Apply the Parasoft intent gate from `docs/workflow/agent-workflow.md` before loading skill cards.
- Use Skill 033 for underspecified service-test authoring.
- Use the explicit direct-routing rules in `docs/skills/skill-index.md` for validation enrichment, request-readiness remediation, single-client intent, and direct generation.
- For server-API-mediated work, apply the runtime prelude first, and re-run routing on every new user prompt.

If intent is still unclear after routing, ask one targeted clarification question.
