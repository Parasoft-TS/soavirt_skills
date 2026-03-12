# Change Advisor Completion Artifact Template

Use this template when producing the final completion artifact for Skill 061.

## A) Refactor Outcome
- **Original source `.tst`:**
- **Copied refactored `.tst`:**
- **Source OpenAPI -> Target OpenAPI:**

## B) Processing Summary
- **Targeted clients processed:**
- **Review items resolved:**
- **Final structural verification result:**
- **Copied artifact outcome:** kept / rolled back
## C) Success Tier Status
- **Structural success:**
- **Mechanical migration success:**
- **Runtime-readiness status:**
- **Semantic-confidence status:**

## D) Follow-Up Hotspots
- **Partial, blocked, deferred, or rolled-back areas:** `<list or none>`

## E) Recommended Next Step
- **Next step:** focused execution and traffic confirmation of changed tests

## Required Output Rules
- Section order is mandatory: `A`, `B`, `C`, `D`, `E`.
- Do not omit a section because a value is missing; emit `Unknown (<reason>)` when necessary.
- Section `D` must appear even when there are no hotspots; emit `none` in that case.
- If chained-tool repair was explicitly deferred, include that in Section `D` rather than implying full completion.
- Section `C` should preserve the distinction between:
  - structural artifact validity
  - mechanical migration outcome across targeted clients
  - runtime-readiness follow-up state
  - semantic-confidence / known-hotspot state
- Keep the artifact concise and completion-oriented rather than reopening the full planning narrative.
