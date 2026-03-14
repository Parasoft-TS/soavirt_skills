# Skill 033 Completion Artifact Template

Use this template when producing the post-execution completion artifact for Skill 033.

## A) Outcome Summary
- **Target `.tst`:** `<name>`
- **Operations processed:** `<method path>, <method path>`
- **Final phase reached:** `<short phase summary>`
- **Overall result:** `<complete | partial | blocked | deferred>`

## B) Work Performed
- **Happy-path:** `<short summary>`
- **Standard negatives:** `<short summary>`
- **Security:** `<short summary>`
- **Validation:** `<short summary>`

## C) Remaining / Follow-up Items
- **Remaining:** `<none or short bullet list>`
- **Recommended next step:** `<short operator-facing next step>`

## Required Output Rules
- Section order is mandatory: `A`, `B`, `C`.
- Keep the completion artifact concise by default.
- For straightforward complete runs, prefer compact bullets or short grouped lines over a large number of subsections.
- Use the same structure for partial or blocked runs, but let Section `C` carry the unresolved or deferred items.
- Do not re-explain internal workflow structure, planner rationale, or layout-detection logic in the default artifact.

## Representative Example (Illustrative)
```text
A) Outcome Summary
- Target `.tst`: Parabank_Subset_Happy_Negative.tst
- Operations processed: POST /deposit, GET /accounts/{accountId}
- Final phase reached: generation + readiness + negative calibration + validation enrichment
- Overall result: complete

B) Work Performed
- Happy-path: configured both selected operations with approved request values
- Standard negatives: created 4 for /deposit and 2 for /accounts/{accountId}
- Security: created security branches for both operations with Penetration Testing Tool attached
- Validation: happy-path and standard negatives enriched in one combined pass; standard negatives validated conservatively; security branches left without additional chained validators

C) Remaining / Follow-up Items
- Remaining: none
- Recommended next step: run the `.tst` end-to-end once and review whether any operation needs expanded negative coverage beyond the first-pass evidence-backed set
```
