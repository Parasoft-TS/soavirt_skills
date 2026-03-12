# Skill Authoring Workflow

## Purpose
Define how contributors and maintainers design, refine, validate, and evolve skills and workflow cards for SOAtest and Virtualize.

## Scope & Audience
- **Primary audience:** contributors/maintainers who create or modify skill cards, routing surfaces, and workflow docs.
- **Primary purpose:** govern how reusable skills should be structured and evolved over time.
- **Not this document's purpose:** live runtime execution policy for user-facing tasks. Runtime-global policy lives in `docs/workflow/agent-workflow.md`.
- **Companion document:** use `docs/workflow/documentation-sync-workflow.md` for canonical doc/log update rules when authoring changes are made.

## Working Principles
- Build from narrow, testable tasks to broader capabilities.
- Establish shared REST skills first, then product-specific skills.
- Keep write operations explicit and validated with round-trip checks.
- Record both execution history (chat log) and rationale (decision log).

## Skill Organization Model
- **Atomic skills (preferred for reuse):** one endpoint + one intent (for example introspect, download, upload, copy).
- **Composite skills:** orchestrate 2-4 atomic skills to complete a common workflow (for example copy + verify + optional rename check).
- **Composite orchestration skills:** user-intent intake + branching flows that coordinate multiple atomic/composite skills across domains.
- **Product overlays:** SOAtest/Virtualize-specific guidance that references atomic/composite shared skills rather than duplicating them.

### Skill Folder Convention
- Keep endpoint-focused atomic skills in domain folders (for example `platform`, `validation`, `client-tools`).
- Keep reusable cross-domain orchestration cards in `docs/skills/composite-orchestration/`.

## Architectural Layering Model
Use the layering model for dependency reasoning and long-term decomposition. Use `docs/skills/skill-index.md` as the primary operator-facing navigation surface.

- **L0 Platform/File Ops:** list, copy, rename, delete, download, upload.
- **L1 Read / Inspect:** summarize, inspect, and trace asset configuration and available data.
- **L2 Structural Mutation:** create, move, reorder, rename, and prune suites/tools/datasources.
- **L3 Configuration Mutation:** edit specific tool/test configuration fields and settings.
- **L4 Semantic / Orchestration:** apply style rules, conventions, and multi-skill orchestration logic.

### Composition Rule
- Composite skills should reference lower-layer skills by id and avoid embedding raw endpoint logic.
- Product-specific cards should point to shared lower-layer cards first, then add only delta behavior.

## Card Design and Context Discipline
- Keep each card short and task-focused; avoid mixed concerns in one card.
- Store canonical examples in the card that owns the operation; other cards should link instead of duplicating.
- Prefer adding a new atomic card when a step introduces a new endpoint, object model, or failure mode.
- Use composite cards only for repeated operator workflows, not for one-off experiments.
- Author every skill card for a future agent that does not have privileged conversational context from the authoring session.
- Do not rely on compressed or implied reasoning when a future agent would need explicit instruction to act consistently; encode ownership boundaries, authoritative evidence, approval gates, and fail-closed limits in the card when they matter to execution.
- Practical self-check: for each major procedure step, ask whether a different agent could perform it reliably from the card alone, or whether the step still depends on context that only the author implicitly remembers.

## Ownership Split: Atomic Skills vs Workflow Orchestration
- Atomic skill cards should define endpoint-accurate execution plus required-input gates for their own writes.
- Composite/workflow cards should own cross-skill questioning, option summarization, and branching decisions.
- If intent branching repeats across sessions, promote it to a reusable composite workflow card rather than duplicating logic in multiple atomic cards.
- Use `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` as the canonical interaction model for service-test authoring intake and staged execution; other orchestration cards should define only branch-specific deltas relative to it.

## Standard Loop (Per Skill)
1. Define the skill card objective, scope, inputs, and preconditions.
2. Validate endpoint behavior with read-only calls.
3. Capture examples and response shapes.
4. Add write capability only when the read path is stable.
5. For write calls, verify request payload encoding/content type before first execution (JSON and YAML must be UTF-8 without BOM unless endpoint contract specifies otherwise).
6. Validate post-conditions and document failure modes.
7. When a skill runs tests, validate with the run-results-traffic triad before concluding diagnostics.
8. Update the canonical docs/logs required by `docs/workflow/documentation-sync-workflow.md`.
9. Link the card in `docs/skills/skill-index.md` with the right operator-facing family and dependency references.

## Suggested Authoring Session Cadence
- Session start: identify the single skill, workflow, or doc surface being changed.
- During session: keep concise action notes in `docs/logs/chat-log.md`.
- Session end: update `docs/logs/decision-log.md` when a reusable policy/architecture decision was made, and update `docs/skills/backlog.md` when status changed.

## Contributor Continuity
For contributor/maintenance sessions, prefer rereading:
- `docs/workflow/skill-authoring-workflow.md`
- `docs/workflow/documentation-sync-workflow.md`
- `docs/skills/skill-index.md`
- `docs/skills/backlog.md`
- `docs/logs/decision-log.md`
