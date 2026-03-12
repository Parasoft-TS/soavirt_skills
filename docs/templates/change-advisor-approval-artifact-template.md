# Change Advisor Approval Artifact Template

Use this template when producing the initial post-analysis approval artifact for Skill 061.

## A) Header
- **Refactor plan for:** `<source tst>`
- **Source OpenAPI:** `<confirmed source anchor>`
- **Target OpenAPI:** `<confirmed target anchor>`
- **Current/source base URL evidence:** `<one value, multiple exact-match values, or none>`
- **Target OpenAPI base URL candidate:** `<value or none>`
- **Base URL policy status:** `<pending review | keep current/source | rewrite to target candidate | rewrite to user-supplied>`

## B) Planning Note
- **Blocked, ambiguous, or explicitly deferred hotspots:** `<short paragraph or none>`

## C) Legend
- `[AUTO]` refactor target can proceed without more user input
- `[REVIEW]` refactor target needs user confirmation
- `[BLOCKED]` refactor target needs user direction before refactor can proceed
- Unmarked nodes are context only and are not refactor targets.

## D) Structural Tree
- **Tree body:**
  - render the selective tree view here
  - use actual suite/scenario/test/tool labels
  - include only final-renderable tree lines
  - preserve stable source-tree order

## E) Grouped Review Packet
- **Review items order:**
  - workflow-global run-policy items first (for example base URL policy)
  - then by target client and dependency order:
    - operation
    - path/query parameters
    - request body
    - chained tools
- **Review item shape:**
  - `R<n> [REVIEW|BLOCKED] <mapping layer or run policy>`
  - `Decision needed:`
  - `Agent proposal:`
  - `Why review is needed:`
  - `Depends on:` when applicable
- **If no review items remain:** emit `none`
- **Reply instruction:** tell the user to answer by review id

## F) Final Approval Question
- **Approval prompt:** ask whether to proceed to the write/preparation phase after the review items are resolved

## Required Output Rules
- Section order is mandatory: `A`, `B`, `C`, `D`, `E`, `F`.
- The initial grouped review packet belongs inside this approval artifact after the structural tree and before the final approval question.
- Do not split the first grouped review packet into a separate first response by default.
- Keep the tree body render-safe:
  - no meta-comments
  - no placeholder instructions inside the tree
  - no angle-bracket guidance inside final tree lines
- By default, omit top-level `Data Sources` and `Environments` from the visible tree unless they are directly needed to explain the migration slice.
- Collapsed sibling branches must render as one unmarked context line using the actual label and name.
- Traffic Viewer remains unmarked context by default.
- Other non-target executable/context branches may appear unmarked when they help explain the surrounding scenario structure.
- When an expanded scenario contains nearby non-target executable context such as a `DB Tool`, prefer showing that node as an unmarked context line rather than omitting it.
- Do not invent extra tree markers beyond `[AUTO]`, `[REVIEW]`, and `[BLOCKED]`; blocked or ambiguous details belong in Section `E` and the approval question, not in ad hoc new marker forms.
- Source OpenAPI verification mismatch should not appear in this artifact; that must already have been resolved or blocked earlier in the intake/verification gate.
- If chained-tool repair was explicitly deferred, Section `B` or Section `E` must say so explicitly rather than implying full downstream completion.
- If there are no blocked or ambiguous hotspots, Section `B` must still appear with `none`.
- If there are no review items, Section `E` must still appear with `none`.

## Representative Example (Illustrative)
This example is explanatory only. It shows the intended rendering shape for the initial approval artifact; it is not an additional required output section.

```text
Refactor plan for: parabank.tst
Source OpenAPI: http://example/parabank/v1/openapi.yaml
Target OpenAPI: http://example/parabank/v2/openapi.yaml
Current/source base URL evidence: http://example/parabank/v1
Target OpenAPI base URL candidate: http://example/parabank/v2
Base URL policy status: pending review
One client has a blocked body-mapping issue that needs user input before the write phase can begin.
Legend:
[AUTO] refactor target can proceed without more user input
[REVIEW] refactor target needs user confirmation
[BLOCKED] refactor target needs user direction before refactor can proceed
Unmarked nodes are context only and are not refactor targets.
[REVIEW] Test Suite: /parabank/services/bank/openapi.yaml
  [REVIEW] Scenario: bill pay scenario
    [AUTO] Test 1: /customers/{customerId}/accounts - GET
      [AUTO] Response Traffic -> JSON Validator
      Traffic Object -> Traffic Viewer
    [REVIEW] Test 2: /billpay - POST
      [REVIEW] Response Traffic -> JSON Data Bank
      [AUTO] Response Traffic -> JSON Validator
      Traffic Object -> Traffic Viewer
    DB Tool: DB Tool
  Test Suite: preserved sibling suite name
Review needed before write phase:
- R1 [REVIEW] Run policy
  Decision needed: choose whether to keep the current/source base URL, rewrite to the target OpenAPI base URL, or provide a new base URL
  Agent proposal: rewrite exact source base URL matches to http://example/parabank/v2
  Why review is needed: the target OpenAPI exposes a new server/base URL, but the workflow must not change runtime targets silently
Test 2: /billpay - POST
- R2 [REVIEW] Operation mapping
  Decision needed: confirm the target operation
  Agent proposal: POST /bill-payment
  Why review is needed: source operation is not present verbatim in target, but one strong replacement candidate exists
- R3 [BLOCKED] Request body
  Decision needed: provide a mapping or value source for new required field `paymentDate`
  Agent proposal: none
  Why review is needed: target schema introduced a required field with no trustworthy autonomous value source
  Depends on: R2
- R4 [REVIEW] JSON Data Bank
  Decision needed: confirm whether extracted field `/confirmation/id` should map to `/payment/id`
  Agent proposal: remap `/confirmation/id` -> `/payment/id`
  Why review is needed: one plausible successor field exists, but selector meaning changed
  Depends on: R2
Reply with approvals, rejections, or replacement mappings by review id.
Proceed to the write/preparation phase after resolving the review items above?
```
