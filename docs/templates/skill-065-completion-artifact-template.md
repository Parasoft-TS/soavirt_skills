# Skill 065 Completion Artifact Template

Use this template when producing the post-execution completion artifact for the explicit experimental exploration-first lane.

## A) Outcome Summary
- **Target `.tst`:** `<name>`
- **Clusters processed:** `<cluster>, <cluster>`
- **Evidence model used:** `exploration-backed design + focused SOAtest verification`
- **Overall result:** `<complete | partial | blocked | deferred>`
- **Required-phase status:** `<happy-path authoring: delivered|partial|blocked ; negative accounting/authoring: delivered|pending behind happy-path|partial|blocked ; security accounting/authoring: delivered|pending behind happy-path|partial|blocked ; validation enrichment: delivered|pending behind happy-path|partial|blocked ; verification: delivered|pending behind happy-path|partial|blocked>`
- **Terminal-stop basis:** `<no next safe in-scope action remained | delegated blocker set exhausted continuation | not applicable for complete>`

## B) Work Performed
- **Exploration and ledgering:** `<short summary>`
- **Happy-path authoring:** `<short summary>`
- **Optional / Deferred:** `<short summary>`
- **Negative accounting/results:**
  - Repeat one bullet per phase-eligible happy-path slice in stable order.
  - `<slice/scenario label>` -> `<delivered with short negative summary | no-credible-family | deferred | blocked>`
- **Security accounting/results:**
  - Repeat one bullet per phase-eligible operation in stable order.
  - `<operation label>` -> `<delivered with short branch summary | no-credible-target | deferred | blocked>`
- **Validation outcomes:**
  - Repeat one bullet per phase-eligible happy-path slice in stable order.
  - `<slice/scenario label>` -> `<delivered | delivered-valid-finding | no-tool exception | validator-blocked-but-assertor/diff-delivered | blocked>`
- **Verification/correction:** `<short summary>`

## C) Remaining / Follow-up Items
- **Remaining:** `<none or short bullet list, including downstream required phases still pending behind unfinished upstream work when applicable>`
- **Blocker aggregation note:** `<none | short summary of the aggregated blocker set that ended continuation>`
- **Recommended next step:** `<short operator-facing next step>`
- **Optional follow-up offers:** `<none or short bullet list>`

## Required Output Rules
- Section order is mandatory: `A`, `B`, `C`.
- Keep the completion artifact concise by default.
- Make it clear that SOAtest execution here was verification-focused rather than baseline discovery.
- If any in-scope required phase ended unfinished, partial, or blocked, Section `A` and Section `C` must make that explicit rather than implying effective completion from earlier success.
- When a run stops before a downstream required phase becomes phase-eligible for some slices/operations, report those branches as pending remaining work rather than as if they were individually attempted and blocked/deferred.
- When a run stops as `partial` or `blocked`, make the terminal-stop basis and any aggregated blocker set explicit so the reader can tell why continuation ended.
- Do not report `complete` when a required in-scope phase still has unresolved blocked/partial work.
- For partial or blocked runs, Section `C` must name the unresolved clusters, slices, operations, or follow-up decisions.
- Negative and security outcomes must stay visible per phase-eligible slice/operation rather than implying broad coverage from one representative branch.
- When surprising negative semantics materially influenced coverage or expected-code calibration (for example invalid or absent-resource probes that still returned `2xx` or empty-success-like bodies), surface that finding briefly in Section `B`.
- When validation surfaced a genuine service/contract defect, report `delivered-valid-finding` explicitly rather than folding that outcome into vague failure prose.
- When follow-up actions are worth suggesting, keep them concise and experimental-lane-specific rather than reopening a generic planning artifact.
