# soavirt_skills

Skill cards and orchestration guidance for LLM-driven automation of Parasoft SOAtest and Virtualize assets through the SOAVirt REST API (v6).

This repository is aimed at users who want to clone it and use it with an LLM client (for example Copilot/Claude-style coding agents) to author, run, and diagnose service tests.
## Audience & Doc Map

This repository serves two audiences:

- **Operators/Users of SOAtest/Virtualize** (run, diagnose, and extend tests with an LLM)
  - Start here:
    - `AGENTS.md` (agent bootstrap and routing behavior)
    - `docs/skills/skill-index.md` (find the right capability)
    - Target skill cards under `docs/skills/`
- **Contributors/Skill Authors** (design, refine, and add skills/policies)
  - Start here:
    - `docs/workflow/agent-workflow.md` (runtime-global execution policy)
    - `docs/workflow/skill-authoring-workflow.md` (how skills/workflow docs should be designed and evolved)
    - `docs/workflow/documentation-sync-workflow.md` (how canonical docs/logs/indexes stay synchronized)
    - `docs/skills/backlog.md` (status and next work)
    - `docs/logs/decision-log.md` (why key policy choices were made)

Use this split to avoid mixing task execution guidance with skill-governance internals during normal operator flows.

## What You Can Do Today

Current capabilities are strongest for SOAtest `.tst` workflows and include:

- Summarize existing `.tst` assets in human-readable form (suite/test organization, intent, with adjustable detail level).
- Perform deep YAML-based `.tst` configuration analysis with structured outputs:
	- Execution Context
	- Step Flow (consumes/produces + embedded validation details)
	- Available Data Sources (family + columns)
- Discover and inspect server-side assets (`children`, `descendants`, metadata reads).
- Transfer and manage files (`download`, `upload`, `copy`, `rename`, `delete`).
- Create test assets from service definitions:
	- empty `.tst`
	- OpenAPI/Swagger
	- WSDL
	- RAML
	- XSD
- Orchestrate service-test authoring from natural language intent:
	- intake and scope confirmation
	- subset operation selection
	- deterministic naming and write guards
	- multi-step execution planning
- Build and modify test structure:
	- create/configure suites
	- prune generated suites to selected subsets
	- move specific datasource types
- Create and manage tool chains:
	- REST Client
	  - unconstrained lifecycle in None mode
	  - validated v1 single constrained lifecycle, including fresh constrained creation and JSON body-bearing create via YAML promotion plus API body normalization
	- DB Tool
	- JSON/XML Assertor
	- JSON/XML Validator
	- JSON/XML Data Bank
	- Diff Tool (JSON/XML/Text/Binary modes)
- Execute tests and collect diagnostics:
	- queue and poll runs
	- fetch XML results reports
	- retrieve traffic for failure analysis

## How To Use With Your LLM Client

1. Clone this repository and open it as a workspace.
2. With any LLM client (Warp, Copilot, Claude, Windsurf, etc.), send an explicit first instruction before your real task:
	- `Load AGENTS.md and its required startup docs, then wait for my task.`
3. Ensure your SOAVirt server is reachable.
4. Provide the server base URL to your agent (or set `SOAVIRT_BASE_URL`).
5. Ask in plain language (examples):
	- `I want to test my service`
	- `Create tests from this OpenAPI and keep only these operations`
	- `Run this .tst and diagnose failures from traffic`
6. The agent should route to the right cards under `docs/skills/` and apply orchestration rules from `AGENTS.md`.

## Key Files

- `AGENTS.md`: required bootstrap and intent routing rules.
- `docs/workflow/agent-workflow.md`: runtime-global execution and safety policies.
- `docs/workflow/skill-authoring-workflow.md`: contributor workflow for designing and evolving skills.
- `docs/workflow/documentation-sync-workflow.md`: contributor workflow for keeping canonical docs/logs in sync.
- `docs/skills/skill-index.md`: operator-facing capability map and routing surface.
- `docs/skills/backlog.md`: validated/in-progress/planned status by skill.
- `docs/reference/api-spec-cache/`: cached OpenAPI references by server key.
- `docs/logs/decision-log.md`: design decisions and rationale history.

## Repository Layout Policy

- Reusable guidance lives in `docs/`.
- Run-specific evidence and transient artifacts live in `work/`.
- Promote only generalized learnings from `work/` into reusable skill cards.

## Notes

- This repo is currently SOAtest-first, with Virtualize coverage expanding over time.
- Skills are designed to be composable and safe in agentic flows, with explicit write gates and post-write verification.
