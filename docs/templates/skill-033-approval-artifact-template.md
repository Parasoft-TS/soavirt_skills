# Skill 033 Approval Artifact Template

Use this template when producing the interactive-mode pre-write approval artifact for Skill 033.

## Preamble (not a numbered section)
- **Defaults applied:** `<comma-separated list or none>`

## A) Scope and Target
- **Target `.tst`:** `<name>`
- **Selected operations:**
  - `<method path>`
  - `<method path>`

## B) Happy-Path Configuration Blueprint
Repeat this block once per selected happy-path test in stable operation order.

### `<method path>`
- **Parent suite:** `<suite label>`
- **Method:** `<HTTP method or protocol method>`
- **Resolved URL:** `<resolved URL>`
- **Resolved path/query values:**
  - `<name>=<value>`
  - `<name>=<value>`
- **Payload source:** `<explicit user input | contract-derived proposal | same-.tst reuse proposal | none>`
- **Payload preview:** `<preview or none>`
- **Readiness status:** `<ready after approval | needs Skill 058 | blocked>`

## C) Negative and Security Coverage Plan
Repeat this block once per selected operation in the same order used in Section `B`.

### `<method path>`
- **Standard Negative Families Planned:**
  - `<family name>`
  - `<family name>`
- **Security Branch Planned:** `<yes|no>`

## D) Review Items / Remediation Hotspots
- Use this section only for issues that genuinely require user direction or approval before the write phase or before a downstream branch can continue.
- If no review items remain, emit `none`.
- Required review-item shape:
  - `R<n> [REVIEW|BLOCKED] <topic>`
  - `Decision needed:`
  - `Agent proposal:`
  - `Why review is needed:`
  - `Depends on:` when applicable
- Required reply instruction:
  - `Reply with approvals, rejections, or replacements by review id.`

## E) Approval Question
- Ask whether to proceed with the exact planned branch once any review items are resolved.
- If Section `D` is `none`, permit a simple approval reply.

## Required Output Rules
- Section order is mandatory: preamble, `A`, `B`, `C`, `D`, `E`.
- Do not invent extra sections.
- Required fields must appear even when the value is `Unknown (<reason>)`.
- Section `B` and Section `C` entries must remain render-stable and must not collapse into prose summaries.
- Section `C` lists named standard-negative families, not target counts.
- Do not include internal structure-detection logic, planner rationale, or a standalone validation-plan section unless that information is needed to explain a review item or blocked condition.
- Review ids belong only in Section `D`.

## Representative Example (Illustrative)
```text
Defaults applied: auto-name, full scope, standard negatives on, security on, validation enrichment on, DB comparison off

A) Scope and Target
- Target `.tst`: Parabank_Subset_Happy_Negative.tst
- Selected operations:
  - POST /deposit
  - GET /accounts/{accountId}

B) Happy-Path Configuration Blueprint
### POST /deposit
- Parent suite: /deposit
- Method: POST
- Resolved URL: /parabank/services/bank/deposit
- Resolved path/query values:
  - accountId=12345
  - amount=100.00
- Payload source: none
- Payload preview: none
- Readiness status: ready after approval

### GET /accounts/{accountId}
- Parent suite: /accounts/{accountId}
- Method: GET
- Resolved URL: /parabank/services/bank/accounts/12345
- Resolved path/query values:
  - accountId=12345
- Payload source: none
- Payload preview: none
- Readiness status: ready after approval

C) Negative and Security Coverage Plan
### POST /deposit
- Standard Negative Families Planned:
  - missing required input
  - invalid format
  - invalid value/range
  - missing resource
- Security Branch Planned: yes

### GET /accounts/{accountId}
- Standard Negative Families Planned:
  - invalid identifier format
  - missing resource
- Security Branch Planned: yes

D) Review Items / Remediation Hotspots
- R1 [REVIEW] Happy-path request value
  - Decision needed: confirm the reusable accountId
  - Agent proposal: use 12345
  - Why review is needed: this value was proposed from same-.tst evidence rather than explicitly supplied by the user
- R2 [REVIEW] Happy-path request value
  - Decision needed: confirm the proposed deposit amount
  - Agent proposal: use 100.00
  - Why review is needed: this value is contract-compatible but not user-provided
Reply with approvals, rejections, or replacements by review id.

E) Approval Question
Proceed with the happy-path configuration above, create the standard negative and security branches shown above, and continue into calibration plus validation enrichment?
```
