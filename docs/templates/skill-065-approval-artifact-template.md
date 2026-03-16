# Skill 065 Approval Artifact Template

Use this template when producing the interactive-mode pre-write approval artifact for the explicit experimental exploration-first lane.

## Preamble (not a numbered section)
- **Experimental lane:** `explicit opt-in confirmed`
- **Defaults applied:** `<comma-separated list or none>`

## A) Scope and Experimental Target
- **Target `.tst`:** `<name>`
- **Selected scope:** `<all operations | subset summary>`
- **Execution posture:** `<interactive | autonomous>`
- **Evidence model:** `direct exploration before SOAtest authoring`
- **Required in-scope phases:** `<happy-path authoring; negative accounting/authoring; security accounting/authoring; validation enrichment; verification | narrowed-scope equivalent>`
- **User-narrowed or explicitly deferred scope:** `<none | short summary>`

## B) Exploration-Backed Cluster Authoring Plan
Repeat this block once per approved cluster in stable order.

### `<cluster label>`
- **Scope contribution:** `<selected operations or cluster summary>`
- **Default authoring path:** `<read-safe-only | owned-fixture-lifecycle | partial-crud-mutation | shared-resource-borrow-restore-later | hybrid-safe-core>`
- **Baseline plan:** `<whole-payload snapshot + targeted extracts | targeted extracts only | read-safe only summary>`
- **Happy-path structure:** `<Baseline Snapshot / Safe Reference Reads / Lifecycle Flow summary>`
- **Request/value status:** `<approved | ready after approval | review needed | blocked>`
- **Request/value notes:** `<none | short summary>`
- **Support prerequisites:** `<none | summary of approved prerequisite support work | approval needed>`
- **Optional / Deferred:** `<none | short summary>`
- **Authoring status:** `<ready after approval | partially blocked | blocked>`

## C) Negative, Security, Validation, and Verification Plan
Repeat this block once per approved cluster in the same order used in Section `B`.

### `<cluster label>`
- **Negative-accounting plan:**
  - Repeat one bullet per eligible happy-path slice in stable order.
  - `<slice/scenario label>` -> `<planned negative family summary | phase-pending behind happy path | no-credible-family candidate | deferred | blocked>`
- **Negative structure default:** `<single-step direct REST Clients under Negative Coverage | grouped bundle required for minimal-prefix/full-sequence negatives | mixed summary>`
- **Security-accounting plan:**
  - Repeat one bullet per eligible operation in stable order.
  - `<operation label>` -> `<planned security branch | phase-pending behind happy path | no-credible-target candidate | deferred | blocked>`
- **Validation plan:**
  - Repeat one bullet per happy-path slice in stable order.
  - `<slice/scenario label>` -> `<schema + content | content only | phase-pending behind happy path | no-tool exception | validator-blocked possibility | blocked>`
- **Focused verification scope:** `<baseline-only | safe-read | happy-path scenario | negative target | security branch | cluster happy path>`

## D) Review Items / Exploration Blockers
- Use this section only for issues that genuinely require user direction or approval before writes, including unresolved request-value/payload approvals and prerequisite-expansion decisions.
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
- Ask whether to proceed with the exact exploration-backed cluster authoring, negative/security branching, validation, and verification plan once any review items are resolved.
- If Section `D` is `none`, permit a simple approval reply.

## Required Output Rules
- Section order is mandatory: preamble, `A`, `B`, `C`, `D`, `E`.
- Do not invent extra sections.
- Section `B` and `C` must stay per-cluster and render-stable rather than collapsing into prose.
- If exploration-backed request values are not yet approved for a cluster, Section `B` and/or Section `D` must make that explicit.
- Keep the evidence model explicit so the user can tell this is the experimental lane.
- Every in-scope required phase must appear explicitly in the artifact as planned, phase-pending, narrowed/deferred, or blocked; do not imply that a required phase is omitted simply because the rest of the cluster plan is ready.
- During pre-write planning, downstream phases that depend on future happy-path authoring should normally remain planned/pending rather than blocked; use `blocked` only when a real blocker already exists before writes begin.
- Negative-accounting and security-accounting entries must stay per eligible slice/operation rather than broad representative summaries.
- When the plan uses single-step negatives, present them as direct negative targets rather than implying extra bundle or `Negative Cases` hierarchy that the card does not require.
- Review ids belong only in Section `D`.
