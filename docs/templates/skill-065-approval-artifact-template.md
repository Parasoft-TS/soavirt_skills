# Skill 065 Approval Artifact Template

Use this template when producing the interactive-mode pre-write approval artifact for the explicit experimental exploration-first lane.

## Preamble (not a numbered section)
- **Experimental lane:** `explicit opt-in confirmed`
- **Defaults applied:** `<comma-separated list or none>`

## A) Scope and Experimental Target
- **Target `.tst`:** `<name>`
- **Selected operations:**
  - `<method path>`
  - `<method path>`
- **Execution posture:** `<interactive | autonomous>`
- **Evidence model:** `direct exploration before SOAtest authoring`

## B) Exploration-Backed Happy-Path Authoring Plan
Repeat this block once per selected operation in stable order.

### `<method path>`
- **Exploration outcome:** `<usable happy path | blocked-auth | blocked-input | probe-exhausted | negative-only-evidence | spec-runtime-mismatch>`
- **Canonical request source:** `<explicit user input | project-context value | OpenAPI example/default/enum | same-run reusable value | synthesized placeholder>`
- **Resolved request summary:** `<short canonical request summary>`
- **Observed response summary:** `<status + payload/media-type summary>`
- **Mutation owner:** `<Skill 020 | Skill 059 | none>`
- **Authoring status:** `<ready after approval | blocked | fallback to Skill 058 proposed>`

## C) Negative, Security, and Validation Plan
Repeat this block once per selected operation in the same order used in Section `B`.

### `<method path>`
- **Standard Negative Families Planned:**
  - `<family name>`
  - `<family name>`
- **Security Branch Planned:** `<yes|no>`
- **Validation Plan:** `<schema + content | content only | no-tool exception | blocked>`

## D) Review Items / Exploration Blockers
- Use this section only for issues that genuinely require user direction or approval before writes.
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
- Ask whether to proceed with the exact exploration-backed authoring, negative/security branching, and validation plan once any review items are resolved.
- If Section `D` is `none`, permit a simple approval reply.

## Required Output Rules
- Section order is mandatory: preamble, `A`, `B`, `C`, `D`, `E`.
- Do not invent extra sections.
- Section `B` and `C` must stay per-operation and render-stable rather than collapsing into prose.
- Keep the evidence model explicit so the user can tell this is the experimental lane.
- Review ids belong only in Section `D`.
