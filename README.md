# soavirt_skills

Skill cards and orchestration guidance for LLM-driven automation of Parasoft SOAtest and Virtualize assets through the SOAVirt REST API (v6).

This repository is aimed at users who want to clone it and use it with an LLM client (for example Copilot/Claude-style coding agents) to author, run, and diagnose service tests.

## What You Can Do Today

Current capabilities are strongest for SOAtest `.tst` workflows and include:

- Summarize existing `.tst` assets in human-readable form (suite/test organization, intent, and evidence-backed detail level).
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
	- DB Tool
	- JSON/XML Assertor
	- JSON/XML Validator
	- JSON/XML Data Bank
	- Diff Tool (JSON/XML/Text/Binary modes)
- Execute tests and collect diagnostics:
	- queue and poll runs
	- fetch XML results reports
	- retrieve traffic for failure analysis

## Current Scope Status

- Validated and actively usable:
	- shared file lifecycle and introspection skills
	- test execution and traffic diagnostics core flow
	- REST Client and DB Tool lifecycle flows
	- `.tst` creation from OpenAPI/WSDL/XSD
	- subset-prune workflow after generated suites
	- cross-cutting safety policies (read-merge-write PUT, capability preflight)
- Defined but still maturing in runtime coverage:
	- some validator/data-bank mode combinations
	- RAML generation runtime validation
	- broader Virtualize-specific workflows

## How To Use With Your LLM Client

1. Clone this repository and open it as a workspace.
2. Ensure your SOAVirt server is reachable.
3. Provide the server base URL to your agent (or set `SOAVIRT_BASE_URL`).
4. Ask in plain language (examples):
	 - `I want to test my service`
	 - `Create tests from this OpenAPI and keep only these operations`
	 - `Run this .tst and diagnose failures from traffic`
5. The agent should route to the right cards under `docs/skills/` and apply orchestration rules from `AGENT.md`.

## Key Files

- `AGENT.md`: Required bootstrap and intent routing rules.
- `docs/workflow/agent-workflow.md`: cross-cutting execution and safety policies.
- `docs/skills/skill-index.md`: capability map by layer and intent domain.
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
