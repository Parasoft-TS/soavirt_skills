# soavirt_skills
Repository to house agent skills for SOAtest/Virtualize

## Workflow Artifacts

- `docs/workflow/agent-workflow.md` - primary process for building skills collaboratively.
- `docs/skills/skill-index.md` - layered map of skills for fast retrieval and context control.
- `docs/templates/tst-human-friendly-summary-template.md` - canonical summary format for explaining `.tst` contents.
- `docs/reference/api-spec-cache/` - local cached OpenAPI docs by server key (base-URL-driven).
- `docs/logs/chat-log.md` - chronological session log.
- `docs/logs/decision-log.md` - key decisions and rationale.
- `docs/skills/skill-card-template.md` - template for defining each skill.
- `docs/skills/backlog.md` - ordered list of planned and in-progress skills.
- `work/README.md` - boundary for run-specific generated artifacts (downloads/snapshots/one-off outputs).

## Server Onboarding (API Docs)

- Provide server base URL (for example `http://<host>:<port>/soavirt/api/v6`).
- Cache the OpenAPI spec under `docs/reference/api-spec-cache/<server-key>/openapi_v6.yaml`.
- Use cached spec as local reference during skill iteration.

## Artifact Boundary (Important)

- `docs/skills`, `docs/templates`, and `docs/workflow` are reusable, general-purpose knowledge.
- `work/` is for task/run-specific artifacts tied to a particular file or server state.
- Summaries generated for a single target asset should be treated as working artifacts unless explicitly promoted into reusable guidance.

## How We Use This Repo

1. Pick one item from the backlog.
2. Create or update a skill card from the template.
3. Validate endpoint behavior and collect examples.
4. Update chat log and decision log at session end.

## Current Initial Focus

Shared REST lifecycle skills applicable to both SOAtest and Virtualize:

- `GET /v6/children`
- `GET /v6/descendants/files`
- `GET /v6/files/download`
- `POST /v6/files/upload`
