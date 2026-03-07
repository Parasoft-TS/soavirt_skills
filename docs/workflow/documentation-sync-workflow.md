# Documentation Synchronization Workflow

## Purpose
Define how canonical docs, logs, indexes, templates, and transient artifacts stay in sync when skills, policies, or information architecture change.

## Scope & Audience
- **Primary audience:** contributors/maintainers updating skill cards, workflow docs, routing surfaces, logs, or repository information architecture.
- **Primary purpose:** keep one clear canonical source for each kind of information and prevent stale proposal/transition artifacts from becoming competing guidance.
- **Not this document's purpose:** live runtime execution policy. Runtime-global policy lives in `docs/workflow/agent-workflow.md`.

## Canonical Document Roles
- `AGENTS.md`
  - runtime bootstrap, startup reading order, and high-level routing/guardrails.
- `docs/workflow/agent-workflow.md`
  - global runtime execution policy, loading rules, safety rules, and failure handling.
- `docs/workflow/skill-authoring-workflow.md`
  - contributor workflow for designing and evolving skills.
- `docs/workflow/documentation-sync-workflow.md`
  - contributor workflow for keeping canonical docs/logs synchronized.
- `docs/skills/skill-index.md`
  - operator-facing routing and navigation surface.
- `docs/skills/backlog.md`
  - status surface for defined, validated, in-progress, planned, retired, or deferred skills.
- `docs/logs/decision-log.md`
  - durable rationale for reusable policy/architecture decisions.
- `docs/logs/chat-log.md`
  - chronological record of what happened in specific build/maintenance sessions.
- `docs/templates/`
  - canonical output contracts and reusable response structure.
- `work/`
  - transient run-specific evidence, experiments, caches, and snapshots.

## Policy Change Synchronization Rule
- When a global execution policy changes in `docs/workflow/agent-workflow.md`, update all affected canonical surfaces in the same session:
  - `AGENTS.md` runtime-critical summary if bootstrap/runtime summary changed,
  - `docs/logs/decision-log.md` with rationale and impact,
  - `docs/skills/skill-index.md` if routing/navigation behavior changed.
- Keep detailed runtime policy canonical in `docs/workflow/agent-workflow.md`; keep `AGENTS.md` concise and bootstrap-oriented.

## Artifact Separation Policy
- Keep reusable knowledge in:
  - `docs/skills/` (skill cards)
  - `docs/templates/` (reusable output formats)
  - `docs/workflow/` (process and policy)
- Keep run-specific files in `work/`:
  - downloaded YAML/JSON snapshots
  - target-specific summaries
  - temporary diffs and experiment outputs
  - caches and ledgers that are session/runtime evidence rather than reusable guidance
- Treat `work/` artifacts as optional transient evidence for reproducibility, not as required input for future skill execution.
- For live runtime tasks, `work/` artifacts may support reproducibility or post hoc comparison, but they do not override canonical runtime targeting and routing rules in `docs/workflow/agent-workflow.md`.
- Do not encode target-specific facts (for example one file's suite names) into reusable skill cards or workflow docs.

## Required Follow-Through by Change Type

### 1) Runtime Policy Changes
When runtime-global behavior changes:
- update `docs/workflow/agent-workflow.md`,
- update `AGENTS.md` if bootstrap/runtime summary changed,
- add a `docs/logs/decision-log.md` entry,
- update `docs/skills/skill-index.md` if routing behavior changed.

### 2) Skill Additions, Retirements, or Major Scope Changes
When a skill is added, retired, renamed, or materially repurposed:
- update the skill card(s),
- update `docs/skills/skill-index.md`,
- update `docs/skills/backlog.md`,
- record meaningful build activity in `docs/logs/chat-log.md`,
- add a decision-log entry when the change introduces reusable policy or architecture, not just status movement.

### 3) Output Contract / Template Changes
When response shape or template rules change:
- update the owning template in `docs/templates/`,
- update all owning skill cards that delegate to that template,
- update `docs/skills/backlog.md` if wording/status becomes stale,
- log the rationale in `docs/logs/decision-log.md` if the change affects reusable behavior or compliance expectations.

### 4) Routing / Taxonomy / Information Architecture Changes
When the project reorganizes routing, taxonomy, or navigation:
- update `docs/skills/skill-index.md` as the live operator-facing surface,
- update `docs/workflow/skill-authoring-workflow.md` if contributor guidance changed,
- update `README.md` if contributor/operator entry points changed,
- add a `docs/logs/decision-log.md` entry to preserve rationale.

### 5) Proposal / Transition Artifact Retirement
When a proposal or transition document is no longer needed:
- confirm its intent is already preserved in canonical docs and/or logs,
- capture any missing rationale in `docs/logs/decision-log.md` and notable implementation history in `docs/logs/chat-log.md`,
- then delete or clearly retire the artifact so it does not compete with canonical guidance.

## Canonical-Source Rule
- Each reusable concept should have one obvious canonical home.
- Workflow docs should point to canonical skill cards, indexes, templates, or logs instead of re-explaining content they do not own.
- If two docs appear to govern the same behavior, consolidate ownership rather than maintaining parallel guidance.
