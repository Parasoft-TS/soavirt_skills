# soavirt_skills

This repository now serves two roles at the same time:

- a Parasoft SOAtest/Virtualize workspace (`ProvisioningAssets`, `TestAssets`, and `VirtualAssets`)
- an agent-skills project that teaches an LLM agent how to work safely and effectively inside that workspace

In short, this repo is a SOAtest/Virtualize workspace plus the agent skills and rules for working in it.

## Getting Started

1. Create a new workspace from SOAtest/Virtualize Desktop. The default location is typically `C:/Users/<user>/parasoft/soavirt_workspace`.
2. In the new workspace, the Project Explorer will usually show `ProvisioningAssets`, `TestAssets`, and `VirtualAssets`. Select all three projects, right-click, and delete them.
   - Leave **Delete project contents on disk (cannot be undone)** unchecked.
3. Import this repository into the workspace:
   - `Import > Git > Projects from Git`
   - choose either **Existing local repository** or **Clone URI**
   - select the repository
   - choose **Import Existing Eclipse Projects**
   - select all projects in the table
   - make sure **Search for nested projects** is checked
4. If you want to edit root-level dotfiles from the Project Explorer, open the three-dot menu in Project Explorer, choose **Filters and Customization...**, and uncheck `.* resources`.
5. Make sure SOAtest/Virtualize Desktop is licensed and the local SOAtest/Virtualize Server is running. A quick check is to open `http://localhost:9080/soavirt/api` and confirm the Swagger page loads.

## Audience

This repository supports two primary workflows:

- **Operators / SOAtest-Virtualize users**
  - use the repo to create, inspect, run, diagnose, extend, and refactor test assets
  - common session bootstrap prompts:
    - `Follow @AGENTS.md and required docs`
    - `Follow @AGENTS.md and required docs for project <project_name>`
- **Contributors / skill authors**
  - use the repo to design, refine, validate, and document the skills/policies themselves
  - common session bootstrap prompt:
    - `Follow @AGENTS.md and required docs for skill contributor workflow`

Keeping those two workflows distinct helps avoid mixing runtime task execution guidance with contributor-only governance and maintenance rules.

## Project-Aware Operator Workflow

When a task depends on application-under-test or project-specific context, the startup flow routes into the project bootstrap skill before broader work continues. In practice, that means the agent can:

- create or repair `TestAssets/project-index.yaml` as the lightweight discovery registry for known projects
- create or update `TestAssets/<project>/<project>.yaml` as the durable per-project record
- create or update project-local supporting folders such as `TestAssets/<project>/environments/`, `data/`, and `resources/`
- keep sensitive values out of tracked docs by defaulting to a gitignored path such as `.soavirt/projects/<project>/secrets.env`
- use the active project as the default anchor for later local asset targeting and project-local output
- let you switch to a different project later in the same session and reload that project's durable context

You can ask the agent to help with SOAtest work such as:

- `Help me test the services in Parabank`
- `Can you add validations to my smoke.tst file?`
- `Can you run regression.tst and tell me why the tests are failing?`
- `Can you summarize what legacy_tests.tst does?`
- `Can you analyze the "Complex Tests" test suite and tell me how it's configured?`
- `Can you help me refactor my parabank_v1.tst tests for a new OpenAPI spec?`

## Contributor / Skill-Author Workflow

After the initial bootstrap, you can switch modes in the same session by telling the agent:

- `I am in skill contributor mode now (see workflows).`
- `I am in SOAtest user mode now (see workflows).`

The repo is designed for progressive context loading: load the startup and workflow surfaces first, then only the specific skill cards, templates, or references needed for the active task.

- `AGENTS.md`
  - startup bootstrap, required reading order, mode split, routing handoff, and high-level guardrails
- `docs/workflow/agent-workflow.md`
  - canonical runtime execution policy, loading rules, safety gates, branch escalation, and failure handling
- `docs/workflow/skill-authoring-workflow.md`
  - contributor workflow for designing, refining, validating, and decomposing skill cards over time
- `docs/workflow/documentation-sync-workflow.md`
  - contributor workflow for keeping canonical docs, logs, indexes, templates, and storage contracts synchronized
- `docs/skills/skill-index.md`
  - operator-facing routing surface and capability map; this is the main entry point for choosing the minimum required skill family
- `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`
  - canonical intake/orchestration surface for underspecified service-test authoring and staged broad-authoring workflows
- `docs/skills/composite-orchestration/skill-061-change-advisor-bulk-openapi-refactor.md`
  - grouped-review, copy-first bulk constrained REST Client OpenAPI refactor workflow for existing `.tst` files
- `docs/skills/composite-orchestration/skill-063-project-context-bootstrap-orchestration.md`
  - durable project-context load/create/update workflow used when the session needs active project context

## What You Can Do Today

Current coverage is strongest for SOAtest `.tst` workflows. Maturity varies by skill card: some paths are validated, while others are defined and still being hardened. Today the repository is strongest at:

- bootstrapping and reusing durable project context under `TestAssets/`
- summarizing existing `.tst` assets and performing deeper YAML-based configuration/dataflow analysis
- resolving local workspace assets safely and performing local file lifecycle work
- creating new test assets from an empty scaffold or from service definitions
  - the most mature generation paths are OpenAPI, WSDL, and XSD
  - RAML support exists but is less validated
- orchestrating broad service-test authoring from natural-language intent, including staged intake and branch selection
- creating and modifying key test structures such as suites, generated subsets, and selected datasource moves
- working with core client, validation, and data-exchange tool chains such as REST Client, DB Tool, Assertors, Validators, Data Bank tools, and Diff Tool workflows
- running tests, collecting XML reports, retrieving traffic, and using those diagnostics to explain failures
- performing copy-first bulk OpenAPI-to-OpenAPI constrained REST refactor workflows on existing `.tst` files

Virtualize-specific coverage exists through the shared platform and policy model, but the repo is still more mature on the SOAtest side today.

## Recommended Reasoning Settings

If your LLM client exposes configurable reasoning depth, use it deliberately:

- **Use high reasoning for composite/orchestration tasks** that must coordinate multiple skill cards, preserve approval gates, or produce template-owned artifacts.
  - Examples:
    - broad service-test authoring and staged planning
    - validation-enrichment orchestration
    - request-readiness remediation across multiple tests
    - bulk OpenAPI-to-OpenAPI constrained REST refactor workflows
    - contributor-mode policy, workflow, or template maintenance
- **Use medium reasoning for narrower atomic or near-atomic tasks** where the intent is already clear and the work stays inside one main skill family.
  - Examples:
    - summarize a `.tst`
    - inspect datasource columns
    - run a known test and fetch diagnostics
    - create or edit one tool with already-specified inputs
    - perform one focused read-only analysis pass

### Practical Rule of Thumb

Prefer **high reasoning** when the task does any of the following:

- spans multiple phases or multiple skill families
- requires grouped review or explicit approval before writes
- depends on fail-closed completeness checks
- must follow a template-owned output contract exactly
- involves ambiguous mapping, migration, or cross-artifact refactoring

Prefer **medium reasoning** when the task is already specific, mostly local, and does not require broad orchestration.

When unsure, start with **high reasoning** for the initial routing/analysis phase, then drop to **medium** for follow-up atomic work if your client supports changing it per prompt.

## Key Files

- `AGENTS.md`: required bootstrap, startup reading, intent-routing, and high-level guardrail surface
- `docs/workflow/agent-workflow.md`: canonical runtime execution and safety policy
- `docs/workflow/skill-authoring-workflow.md`: contributor workflow for designing and evolving skills
- `docs/workflow/documentation-sync-workflow.md`: contributor workflow for keeping canonical docs and logs aligned
- `docs/skills/skill-index.md`: operator-facing capability map and routing surface
- `docs/skills/backlog.md`: validated, defined, in-progress, planned, deferred, and retired skill status
- `TestAssets/project-index.yaml`: lightweight registry of durable project records; created or repaired as part of project bootstrap
- `TestAssets/<project>/`: canonical per-project root for durable project context, environment files, data, resources, and default project-local outputs
- `docs/reference/api-spec-cache/`: cached OpenAPI references by server key
- `docs/logs/decision-log.md`: durable design and policy rationale history

## Repository Layout Policy

- reusable guidance lives in `docs/`
- durable project context lives under `TestAssets/project-index.yaml` and `TestAssets/<project>/`
- run-specific evidence and transient artifacts live in `work/`
- gitignored local sensitive project files may live under `.soavirt/`
- promote only generalized learnings from `work/` into reusable skill cards or workflow docs

## Notes

- this repo is currently SOAtest-first, with broader Virtualize coverage still expanding
- the skills system is designed around progressive loading, explicit write gates, and fail-closed safety checks
