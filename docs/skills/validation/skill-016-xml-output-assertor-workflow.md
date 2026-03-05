# Skill 016: XML-Output Assertor Lifecycle (Cross-Tool)

## 1) Skill Name
Add, modify, read, delete, copy, and validate XML Assertor on XML-producing tool output

## 2) Objective
Create a reusable lifecycle pattern for XML Assertors on tool output and validate live XPath assertions against runtime traffic/output.

## 3) Scope
- In scope:
  - create XML Assertor via `POST /v6/tools/xmlAssertors`
  - configure assertions via `PUT /v6/tools/xmlAssertors?id=...`
  - readback and verify configuration via `GET /v6/tools/xmlAssertors?id=...`
  - delete XML Assertor via `DELETE /v6/tools?id=...`
  - copy XML Assertor via `POST /v6/tools/copy`
  - run focused execution and validate runtime behavior via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - service/database data remediation
  - UI-only assertion editing workflows

## 4) Inputs
- Required:
  - target output-provider parent id
  - XML Assertor name
  - assertion XPath and expected value
- Optional:
  - `trimContent` option
  - execution filter (`testNames`) for focused runtime checks
  - copy target parent/name

## 5) Preconditions
- API reachable and authenticated.
- Target test/tool produces XML output at runtime.
- Parent id resolves to an output-provider location that accepts assertor children.
- Runtime response media type for target output is XML (confirmed from baseline run evidence).

## 6) Procedure
1. Construct the output-provider parent path for the target producer tool:
  - REST Client: `<rest-client-id>/Response Traffic`
  - DB Tool: `<db-tool-id>/Results as XML`
  - Do NOT pass the producer tool id directly as `parent.id`; use the output-provider path.
  - See `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` Section 5 for canonical patterns.
2. Run baseline execution first and observe live XML payload shape/content from runtime traffic.
  - if observed payload is not XML, stop XML Assertor flow and route validation intent to the appropriate tool family:
    - JSON payload -> Skill 010/029 family
    - plain text payload -> Skill 031 Diff Tool in text mode.
  - if observed payload is XML, select between XML Assertor and Diff Tool XML mode based on response data volatility:
    - significant dynamic/volatile fields (timestamps, generated ids, session tokens) -> prefer XML Assertor with targeted XPath assertions on stable fields,
    - mostly static response data -> prefer Diff Tool XML mode for simpler full-response comparison (see Skill 031),
    - when unsure, start with Diff Tool; switch to XML Assertor if the ignored-differences list grows large.
3. Create XML Assertor:
   - `POST /v6/tools/xmlAssertors`
   - payload must include `parent.id` and typically `name`.
4. Configure a live value assertion:
   - `PUT /v6/tools/xmlAssertors?id=<assertor-id>`
   - include `toolSettings.assertions[]`, for example:
     - `type: valueAssertion`
     - `selectedElement.xpath`
     - `configuration.expectedValue` (`type: fixed`, `fixed: ...`)
5. Read back assertor to confirm persisted settings.
6. Optional copy flow:
  - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
  - verify copied assertor via `GET /v6/tools/xmlAssertors?id=<new-id>`.
7. Optional delete flow:
  - `DELETE /v6/tools?id=<assertor-id>` and verify absence in descendants/children.
8. Execute focused verification run and inspect runtime results:
   - if needed, request XML report using `includeXmlReport=true`.
9. If XML parser errors occur (for example `Content is not allowed in prolog`), treat it as input-binding mismatch and re-check the chosen output-provider/input source.

## 7) Validation
- Creation/update/readback all return `200` on valid payloads.
- Readback contains expected assertion payload:
  - `toolSettings.assertions[].type = valueAssertion`
  - expected XPath and value are present.
- Runtime result confirms assertion behavior on live output (pass/fail with diagnosable message).

## 8) Failure Modes
- `400` for malformed assertion payload.
- `404` for invalid parent/id.
- Runtime XML parse failure indicates non-XML input binding, even when assertion XPath is valid.
- Media-type mismatch risk when XML Assertor is attached to JSON/plain-text traffic.
- copy placement/name collisions can produce unintended target placement unless readback is performed.
- Over-softening risk: relaxing strict assertion intent after first failure can hide real regressions.

## 8.1) Failure Handling Rule (No Ignore Branch)
- XML Assertor has no ignored-differences feature and should remain strict by default.
- On assertion failure:
  1. summarize mismatch details in human-readable form (XPath, expected vs actual),
  2. confirm with user whether assertion intent or expected value should change,
  3. route deeper runtime diagnosis to:
     - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
- Do not auto-weaken assertions without explicit user confirmation.

## 9) Safety / Rollback
- Write skill (adds/modifies assertor tool nodes).
- Rollback:
  - remove assertor via `DELETE /v6/tools?id=<assertor-id>`
  - or restore previous config using saved GET snapshot.

## 10) Reuse Notes
- Applies to SOAtest: Yes (validated).
- Applies to Virtualize: Not yet validated.
- Works best when input-binding path is explicitly verified from runtime artifacts.
- Cross-cutting dependency:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - Select semantic output-provider parents (for example response/results) before assertion tuning.
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
  - Use GET -> mutate -> PUT when editing existing assertors to preserve non-target settings.

## 11) Validation Snapshot (2026-03-03)
- Created under DB Tool output provider:
  - `/TestAssets/Basic_Test_Construction_Learning.tst/Test Suite Edited/DB Tool - Root Between Test And Child/Results as XML/XML Assertor - DB Row ID`
- Assertion configured:
  - XPath: `/results/resultSet/rows/row/ID`
  - Expected value: `12345`
- Focused execution after placement under `Results as XML`:
  - summary: `failureCount=2`, `testRunCount=3`
  - decoded XML report confirms DB Tool test status: `pass`
  - prior parser error `Content is not allowed in prolog` is absent.
- Evidence folder:
  - `work/runs/2026-03-03/tst-current/db-tool-root-traffic-run/`