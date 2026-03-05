# Agent Skill-Building Workflow

## Purpose
Define a repeatable way to collaboratively build agent skills for SOAtest and Virtualize.

## Working Principles
- Build from narrow, testable tasks to broader capabilities.
- Establish shared REST skills first, then product-specific skills.
- Keep write operations explicit and validated with round-trip checks.
- Record both execution history (chat log) and rationale (decision log).

## Artifact Separation Policy
- Keep reusable knowledge in:
	- `docs/skills/` (skill cards)
	- `docs/templates/` (reusable output formats)
	- `docs/workflow/` (process and policy)
- Keep run-specific files in `work/`:
	- downloaded YAML/JSON snapshots
	- target-specific summaries
	- temporary diffs and experiment outputs
- Treat `work/` artifacts as optional transient evidence for reproducibility, not as required input for future skill execution.
- Do not encode target-specific facts (for example one file's suite names) into reusable skill cards.

## Skill Organization Model
- **Atomic skills (preferred for reuse):** one endpoint + one intent (for example introspect, download, upload, copy).
- **Composite skills:** orchestrate 2-4 atomic skills to complete a common workflow (for example copy + verify + optional rename check).
- **Composite orchestration skills:** user-intent intake + branching flows that coordinate multiple atomic/composite skills across domains.
- **Product overlays:** SOAtest/Virtualize-specific guidance that references atomic/composite shared skills rather than duplicating them.

### Skill Folder Convention
- Keep endpoint-focused atomic skills in domain folders (for example `platform`, `validation`, `client-tools`).
- Keep reusable cross-domain orchestration cards in `docs/skills/composite-orchestration/`.

## Layered Skill Taxonomy (for Long-Term Refactoring)
- **L0 Platform/File Ops:** list, copy, rename, delete, download, upload.
- **L1 Asset Ops:** parse asset metadata, identify top-level suites/tests, validate structure.
- **L2 Structural Refactors:** suite/test reordering, rename tools/tests/suites, move nodes.
- **L3 Configuration Refactors:** edit specific tool/test configuration fields (high granularity).
- **L4 Semantic/Policy Refactors:** apply style rules, conventions, or broad migration patterns.

### Composition Rule
- Composite skills should reference lower-layer skills by id and avoid embedding raw endpoint logic.
- Product-specific cards should point to shared lower-layer cards first, then add only delta behavior.

### Context Window Control Rules
- Load only the minimum set of cards for a task: one target card + direct dependencies.
- When a skill lists cross-cutting dependencies, load them before executing the skill's procedure. If two skills address the same domain (for example JSON validation tools), load ALL listed dependencies from BOTH skills before proceeding.
- Keep each card short and task-focused; avoid mixed concerns in one card.
- Store canonical examples in the card that owns the operation; other cards link instead of duplicating.
- Prefer adding a new atomic card when a step introduces a new endpoint, object model, or failure mode.
- Use composite cards only for repeated operator workflows (not for one-off experiments).

### Decision Rule
- If the operation can be done entirely server-side with a dedicated endpoint, prefer that endpoint.
- Use download/edit/upload only when file content must actually change.
- For any JSON write request (`POST`/`PUT` with `application/json`), require UTF-8 **without BOM** payload bytes.
	- Avoid PowerShell 5.1 patterns that emit BOM for request files.
	- Prefer in-memory JSON body generation or explicit no-BOM writes before `--data-binary @file`.
- For chained tools, select parent output by semantic payload role first (response/results), not by nearest visible child.
- Treat `Traffic Object` as diagnostics-first output (primarily for `Traffic Viewer`) unless a task explicitly requires chaining a tool to the Traffic Object.
- Before any chaining write operation, consult `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`.
- After validating a new tool/output behavior, update the cheat-sheet in the same session.
- For any test-run analysis, apply the execution triad by default: run (`/testExecutions`) -> results (`/results?includeXmlReport=true`) -> traffic (`/traffic?entityId=...`).
- For traffic calls, require `entityId` and use suite id to discover `trafficViewers[]`, then call the specific output-tool viewer id for focused evidence.
- Before multi-step write orchestration, run a capability preflight for the selected branch (see `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`) and prefer verified-supported endpoint-method pairs over optimistic assumptions.
- Keep compatibility guidance in skills/workflow documents (for example cross-cutting cards), not in `docs/reference/api-spec-cache/` overlays.
- For assertion authoring, compose logic from the user prompt at runtime (field/rule/source), and avoid codifying domain-specific fixed assertion recipes as mandatory patterns.
- For Diff/Assertor stabilization branches, summarize runtime differences in human-readable form (field/XPath/property and expected vs actual) and require explicit user confirmation before adding ignore rules.
- Keep reusable skill cards service-agnostic: do not encode target-specific business preconditions (for example environment data state, account balance assumptions, or vendor-specific runtime quirks) as mandatory steps.
- Capture target-specific preconditions and quirks in transient run artifacts, and only promote them to reusable skills when they generalize across services.
- Keep transient-artifact policy centralized in this workflow doc; avoid repeating `work/` persistence instructions in individual skill cards unless a true exception exists.
- Build new tool configurations from endpoint contracts and skill rules first; existing workspace tool instances may be used only as optional verification, never as a required source template.

### Ambiguous Write Recovery Policy (Global)
- For any write call (`POST`/`PUT`/`PATCH`/`DELETE`), if command output is truncated, missing, or otherwise ambiguous, do not retry immediately.
- First run deterministic readback reconciliation for the same target and intended effect.
- Retry is allowed only when reconciliation confirms the intended write did not occur.
- For create flows, treat "artifact found by exact id/name after ambiguous write" as success and continue downstream steps.
- Before any create retry, run a same-name collision read check and branch to safe handling (reuse/delete-and-recreate/new-name) when the artifact already exists.

### User-Intent Orchestration Method (Required for Multi-Step Flows)
- Use this method when user intent must be translated into multiple endpoint-level actions (for example generation + selection + downstream configuration).
- Phase 1: Intake
	- resolve required inputs via precedence: explicit request -> prior confirmed session value -> environment variable,
	- apply confidence gate; if missing/ambiguous, ask targeted question and wait.
- Phase 2: Summarize Choices
	- produce a concise, human-readable option summary from discovered/runtime data (for example operation list, response types, object candidates),
	- ask user to confirm full vs subset scope and preferred selection mode.
- Phase 3: Plan + Execute
	- map confirmed intent to deterministic skill calls,
	- run writes only after required inputs and intent choices are confirmed,
	- perform verification run/readback and report outcome.

### Conversational Intake Policy (Global)
- Default to conversational intake and free-text interpretation over rigid multiple-choice widgets.
- For subset scope requests, always print a human-readable operation catalog before asking for selection:
	- show method + path (+ short summary when available),
	- allow natural-language selection (for example "all /customers endpoints", "login and transfer").
- Normalize user free-text to explicit selected operations and echo the resolved subset before writes.
- Use selector widgets only when strictly required by client constraints; when used, still provide a conversational interpretation path.

### Approved Plan Completion Policy (Global)
- After the user approves an execution plan, continue through all approved stages without repeated proceed/stop prompts.
- Treat runtime business/test failures as diagnostics outputs unless they create a true dependency blocker.
- Only interrupt for user direction when blocked by missing required inputs or unrecoverable dependency failures (for example unresolved ids, invalid parent type with no safe fallback).
- Report residual failures at the end as outcomes, not as implicit orchestration stop conditions.

### Ownership Split: Atomic Skills vs Workflow Orchestration
- Atomic skill cards should define endpoint-accurate execution plus required-input gates for their own writes.
- Composite/workflow cards should own cross-skill questioning, option summarization, and branching decisions.
- If intent branching repeats across sessions, promote it to a reusable composite workflow card rather than duplicating logic in multiple atomic cards.

## Standard Loop (Per Skill)
1. Define skill card (objective, scope, inputs, preconditions).
2. Validate endpoint behavior with read-only calls.
3. Capture examples and response shapes.
4. Add write capability only when read path is stable.
5. For write calls, verify request payload encoding/content type before first execution (JSON and YAML must be UTF-8 without BOM unless endpoint contract specifies otherwise).
6. Validate post-conditions and document failure modes.
7. When a skill runs tests, validate with the run-results-traffic triad before concluding diagnostics.
8. Update logs and backlog status.
9. Link the card in `docs/skills/skill-index.md` with layer and dependencies.

## First Endpoint Set
- `GET /v6/children`
- `GET /v6/descendants/files`
- `GET /v6/files/download`
- `POST /v6/files/upload`

## Suggested Session Cadence
- Session start: choose one skill card.
- During session: keep concise action notes in chat log.
- Session end: update decision log and backlog.

## Session Continuity
- Prefer a fresh re-intake of canonical docs at session start:
	- `docs/workflow/agent-workflow.md`
	- `docs/skills/skill-index.md`
	- `docs/skills/backlog.md`
	- `docs/logs/decision-log.md`
- Optionally review latest transient run artifacts under `work/runs/<date>/<run-name>/` when runtime context is relevant.
