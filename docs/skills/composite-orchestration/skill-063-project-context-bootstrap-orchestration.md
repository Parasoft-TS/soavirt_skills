# Skill 063: Project Context Bootstrap Orchestration (Composite)
## 1) Skill Name
Identify, load, create, repair, update, and persist durable application project context before project-aware application-under-test work begins.
Conversation-first default: ask short natural-language questions and keep stored context compact enough for progressive loading.
## 2) Objective
- Resolve the active application-under-test project immediately after `AGENTS.md` startup loading so follow-on SOAtest/Virtualize work begins with project context already in scope.
- Reuse durable project-specific context when the project is already known.
- Create, repair, or update durable project records when it is not.
- Keep later skills on the load path `TestAssets/project-index.yaml -> TestAssets/<slug>/<slug>.yaml -> environment file(s) -> referenced files only as needed`.
## 3) Scope
- In scope:
  - session-start project identification immediately after `AGENTS.md`/bootstrap required reading
  - deterministic existing-project matching against `TestAssets/project-index.yaml`
  - ambiguous-match disambiguation
  - new-project interview and normalization
  - environment-aware service-definition intake and disambiguation
  - service-definition readback and `BASE_URL` extraction/confirmation
  - existing-project update flow
  - silent repair of unambiguous registry drift
  - persistence under `TestAssets/`
  - canonical project environment-file location and reference rules
  - explicit sensitive-data storage choice with a gitignored default path
  - compact loaded/created/updated summaries using `docs/templates/project-bootstrap-context-summary-template.md`
  - read-time loading contract for future skills
- Out of scope:
  - contributor/skill-authoring workflow storage branches
  - direct SOAtest/Virtualize runtime execution
  - automatic plaintext secret persistence without explicit user approval
## 3.1) Dependencies
- Required:
  - `docs/templates/project-bootstrap-context-summary-template.md`
- Additive:
  - `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
- Canonical references:
  - `AGENTS.md`
  - `docs/workflow/agent-workflow.md`
  - `TestAssets/project-index.yaml`
## 3.2) Ownership Boundaries
- `AGENTS.md` owns the startup hook and required-reading reminder that hands off to this card immediately after startup reading for sessions that loaded `AGENTS.md`.
- `docs/workflow/agent-workflow.md` owns when bootstrap runs and how progressive loading is preserved.
- `docs/skills/skill-index.md` owns routing to this skill.
- This card owns prompts, matching, repair/update rules, persistence, and summary behavior for application-under-test project context.
- Skill 064 owns external environment-file authoring mechanics, internal/referenced environment mutation, and active-environment verification once bootstrap has established the semantic project environment context and canonical file locations.
## 4) Inputs
- Required:
  - a project name, project hint, or permission to ask for one
- Optional:
  - environment details
  - service-definition references
  - durable project notes, facts, or external file references
  - optional application-source-repo reference
  - secret material only when the user intentionally shares it
## 5) Preconditions
- `TestAssets/` is writable.
- If the user approves the default sensitive-data path, `.gitignore` must protect `.soavirt/projects/` before any secret file is written.
- This skill runs immediately after the required startup reading for any session under this repo that has loaded `AGENTS.md`.
## 6) Procedure
### 6.1 Session-start entry
1. If the user already named the project, use that as the initial match input.
2. Otherwise ask:
   - `Tell me what project you're working on so I can load or create the right project context before we continue.`
3. If `TestAssets/project-index.yaml` does not exist:
   - if an unambiguous canonical project root already exists, silently recreate or repair the registry
   - otherwise create the minimal registry shape:
```yaml
schema_version: 1
projects: []
```
### 6.2 Matching inputs and normalization
4. Normalize user input and candidate fields by:
   - lowercasing
   - trimming surrounding whitespace
   - collapsing repeated internal whitespace
   - treating spaces, hyphens, and underscores as equivalent separators
   - removing surrounding punctuation
5. Match only against these registry fields in v1:
   - `slug`
   - `name`
   - `aliases`
6. Do not match against deeper project facts or environment-variable values in v1.
### 6.3 Match outcomes
7. Single clear match:
   - load only the selected project record
   - do not preload all referenced files
   - render the compact summary using `docs/templates/project-bootstrap-context-summary-template.md`
   - then ask:
     - `I loaded in the project data for <project>. Has anything changed, or should we get going?`
8. Ambiguous matches:
   - show at most the top three candidates
   - ask exactly:
     - `I found a few close project matches: <candidate-list>. Which one should I use?`
9. No match:
   - ask exactly:
     - `I don't have a reference to that project. Is this a new project?`
### 6.4 New-project interview
10. After the user confirms the project is new, keep the intake sequence in this order.
11. Ask:
   - `Can you tell me what Environment(s) your application is deployed in for testing?`
12. After environments are established, ask about service definitions using the environment count:
   - if exactly one environment is known, ask:
     - `To help me familiarize myself with your project, do you have any service definitions (OpenAPI/WSDL) associated with the application you want to test?`
   - if multiple environments are known, ask:
     - `To help me familiarize myself with your project, do you have any service definitions (OpenAPI/WSDL) for those environments? If so, which definition goes with which environment?`
13. If the environment-to-definition mapping is ambiguous, ask one targeted clarification question before continuing.
14. Read each provided service definition that is reachable before storing it.
15. Extract `BASE_URL` candidates from each definition and map them to the associated environments.
16. Confirm the extracted `BASE_URL` mapping with the user before persistence.
17. Then ask:
   - `Is there any other information I should know to help me interact with the services of your application (for example login credentials, API keys, or important business data like key ID values)?`
18. Repeat with:
   - `Anything else you'd like me to store about this project before we get going?`
19. Stop only when the user says there is nothing more to add for now.
20. Use the user's confirmed project name as the canonical display name.
21. Generate the slug from the confirmed name unless the user provided a better stable slug explicitly.
22. Capture aliases only when the user provides alternate names.
23. Proactively create or confirm:
   - `TestAssets/<slug>/`
   - `TestAssets/<slug>/environments/`
   - `TestAssets/<slug>/data/`
   - `TestAssets/<slug>/resources/`
### 6.5 Existing-project update branch
24. If the user says the loaded project changed, continue with:
   - `What changed since the last time this project data was updated?`
25. If the answer is broad or unclear, narrow by category:
   - environments
   - service definitions
   - credentials/secrets
   - business data
   - file references
   - other notes
26. After each confirmed update batch, ask:
   - `Anything else changed that I should store before we get going?`
27. Stop when the user says no, persist the updates, summarize what changed with the template, then continue to the working request.
### 6.6 Secret-storage decision
28. When the user provides sensitive data and persistence is in scope, do not silently store it in tracked docs.
29. Use this exact prompt:
   - `You shared information that may be sensitive. I recommend storing it in a gitignored file in this workspace at .soavirt/projects/<project-slug>/secrets.env so it is available to future sessions without being committed. I can also avoid saving it, or save it somewhere else you choose. What would you like me to do?`
30. If the user accepts the default path:
   - ensure `.gitignore` protects `.soavirt/projects/`
   - store the secret only in that local file
   - reference the path from the project record; do not duplicate the secret value there
31. If the user declines persistence:
   - do not write the secret anywhere
   - note in the summary that secret persistence was declined
32. If the user chooses a different path:
   - follow that choice
   - record the path reference in the project record
### 6.7 Durable storage contract
33. Use `TestAssets/` as the canonical durable store.
34. Use this structure:
   - `TestAssets/project-index.yaml`
   - `TestAssets/<slug>/<slug>.yaml`
   - `TestAssets/<slug>/environments/`
   - `TestAssets/<slug>/data/`
   - `TestAssets/<slug>/resources/`
35. Keep `<slug>.yaml` intentionally small and stable so future agents can load it first.
36. Store reference metadata first for external artifacts such as OpenAPI, WSDL, CSV, or Excel files; do not import large summaries by default.
### 6.8 Canonical record shapes
37. `project-index.yaml` should stay discovery-focused and small.
Example:
```yaml
schema_version: 1
projects:
  - slug: parabank
    name: Parabank
    aliases: []
    entry: TestAssets/parabank/parabank.yaml
```
38. `<slug>.yaml` should use stable sections such as:
   - `schema_version`
   - `project`
   - `environment_files`
   - `services`
   - `facts`
   - `references`
   - `notes`
   - optional application-source-repo reference when useful
   - `maintenance`
### 6.9 Environment-file location and handoff rules
39. Treat environment-specific connection details as first-class intake data.
40. When service definitions are provided, read them before storing environment context, extract candidate `BASE_URL` values, and persist `BASE_URL` only after the user confirms the environment mapping.
41. Default environment file location:
   - one environment -> `TestAssets/<slug>/environments/<environment>.env`
   - multiple environments -> `TestAssets/<slug>/environments/<slug>.envs`
42. If a project starts with a single-environment `.env` and later gains additional environments, convert it to `<slug>.envs` while preserving the original environment.
43. Persist the canonical file path/reference in the project record here, but route the actual external environment-file authoring/update mechanics through Skill 064 so this card does not become the owner of XML shape details, `.tst` reference-node mutation, or active-environment verification.
### 6.10 Persistence steps
44. Create or update `TestAssets/<slug>/<slug>.yaml`.
45. Update `TestAssets/project-index.yaml` with discovery fields:
   - `slug`
   - `name`
   - `aliases`
   - `entry`
46. Keep maintenance metadata current after each create or update.
### 6.11 Read-time loading contract for later skills
47. Future project-aware flows should load context in this order:
   - `TestAssets/project-index.yaml`
   - selected `TestAssets/<slug>/<slug>.yaml`
   - referenced `.env` / `.envs`
   - only the specific referenced files needed for the current task
48. Active project context should be the default routing anchor for follow-up project-local asset references.
49. When active-project-local search fails, broaden silently to other projects before concluding that resolution failed.
## 7) Validation
- `TestAssets/project-index.yaml` is valid YAML and contains unique `slug` values.
- Every `entry` path in the registry points to an existing `<slug>.yaml`.
- `<slug>.yaml` remains parseable and follows the stable top-level sections above.
- If service definitions were provided, each stored definition reference has an explicit environment association or an explicitly confirmed shared-environment scope.
- If `BASE_URL` is written for an environment, it was confirmed by the user for that environment before persistence.
- If an environment file is referenced, the path exists under `TestAssets/<slug>/environments/` and matches the `.env` versus `.envs` naming rules.
## 8) Failure Modes
- Ambiguous project names trigger premature auto-selection instead of clarification.
- Registry drift that is clearly repairable is surfaced as user work instead of silently repaired.
- The agent stores service-definition references or `BASE_URL` values without reading the definition and confirming the environment mapping.
- The agent keeps using the superseded `docs/projects/` storage model or older `project.yaml` wording instead of the `TestAssets/` contract.
- The agent carries contributor/skill-authoring workflow into this project-storage branch.
- Sensitive values are copied into tracked docs without explicit approval.
## 9) Exit Condition
- Bootstrap or update ends only when project match/create/repair/update work is resolved and the user says there is no more information to add for now.
- After exit, continue to the user's working request with the resolved project context available for downstream skill loading.
