# Project Bootstrap Context Summary Template
Purpose: provide one compact reusable output contract for Skill 063 when project context is loaded, created, or updated.

## Required fields
- `Mode:` `loaded-existing` | `created-new` | `updated-existing`
- `Project:` `<display name> (<slug>)`
- `Entry:` `TestAssets/<slug>/<slug>.yaml`
- `Environment files:` one compact line or `none`
- `Sensitive storage:` explicit path/decision or `none`
- `Services/definitions:` short bullet list or `none`
- `Changes stored:` short bullet list or `none`
- `Next question:` the exact branch-specific follow-up prompt

## Format
Use this exact section order:
1. `Project context <loaded|created|updated>.`
2. `Project: ...`
3. `Entry: ...`
4. `Environment files: ...`
5. `Sensitive storage: ...`
6. `Services/definitions: ...`
7. `Changes stored: ...`
8. `Next question: ...`

## Rules
- Keep the summary compact; do not dump full project-record contents.
- Show only the environment-file paths, not every variable, unless the user asked for detail.
- When no service definitions are known yet, emit `Services/definitions: none`.
- When the branch only loaded an existing project and nothing changed, emit `Changes stored: none`.
- When the user declined secret persistence, say so explicitly rather than omitting the field.
- The `Next question` line must match the active branch wording owned by Skill 063.

## Example
Project context loaded.
Project: Parabank (`parabank`)
Entry: `TestAssets/parabank/parabank.yaml`
Environment files: `TestAssets/parabank/environments/parabank.envs`
Sensitive storage: `.soavirt/projects/parabank/secrets.env` (gitignored)
Services/definitions:
- `customer-api` -> OpenAPI at `/path/to/customer-api.yaml`
Changes stored: none
Next question: `I loaded in the project data for Parabank. Has anything changed, or should we get going?`
