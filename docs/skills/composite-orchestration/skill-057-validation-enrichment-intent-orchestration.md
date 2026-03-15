# Skill 057: Validation-Enrichment Intent Orchestration (Composite)

## 1) Skill Name
Orchestrate validation enrichment for existing or newly generated service tests.

## 2) Objective
Convert direct validation-enrichment requests (for example "add validations to these tests" or "add assertions to this DB Tool resultset") into a confirmed, executable enrichment plan that uses live runtime evidence, conservative negative-test defaults, explicit schema-source confirmation, and deterministic handoff to the validation, databank, DB, and execution skills.

Conversation-first default: favor natural-language solicitation and interpretation over rigid UI choice widgets when collecting enrichment scope and preference input.

## 3) Scope
- In scope:
  - direct validation-enrichment intent on existing or newly generated service tests and existing DB Tool result outputs
  - resolve selected `.tst` / suite / test scope for enrichment
  - require or collect trustworthy live runtime evidence before proposing validation bundles
  - detect when selected tests or DB Tool outputs are not yet meaningfully configured and hand off to request-readiness remediation before continuing
  - propose happy-path API validation bundles from observed response traffic:
    - schema validator, plus
    - content validation (Diff Tool or Assertor)
  - propose direct DB Tool resultset validation from observed semantic DB output
  - conservative negative-test validation defaults and mismatch surfacing
  - explicit schema-source confirmation, including environment-variable-backed candidates when clearly discoverable
  - response-vs-database consistency orchestration
  - deterministic handoff to downstream validation/data-exchange/DB/execution skills
  - staged execution only after explicit user confirmation
- Out of scope:
  - broad service-test generation or initial authoring intake
  - replacing endpoint-accurate behavior in atomic validation/client cards
  - inventing schema/service-definition sources
  - inventing SQL strategy or DB-side setup/remediation
  - making repeated execution passes to prove response volatility in v1
  - owning primary negative-test generation policy or expected-response-code authoring for new negative variants

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- Additive:
  - `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`
  - `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`
  - `docs/skills/validation/skill-010-json-assertor-workflow.md`
  - `docs/skills/validation/skill-016-xml-assertor-workflow.md`
  - `docs/skills/validation/skill-029-json-validator-workflow.md`
  - `docs/skills/validation/skill-030-xml-validator-workflow.md`
  - `docs/skills/validation/skill-031-diff-tool-workflow.md`
  - `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`
  - `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-051-datasource-introspection-column-discovery.md`

## 4) Inputs
- Required:
  - user objective to enrich existing or newly generated tests or DB Tool outputs with validation
  - target scope anchor:
    - `.tst` file, and/or
    - suite/test/tool selection, and/or
    - explicit test names, tool names, or ids
- Optional:
  - exact user-specified checks
  - approval to use AI-proposed checks as the default baseline
  - WSDL/XSD/OpenAPI source for schema validation
  - response-vs-database comparison intent
  - known DB connection/query/correlation constraints
  - prior execution ids or known recent execution evidence

## 5) Preconditions
- API reachable and authenticated for all downstream atomic skills.
- Target tests or DB Tool outputs are resolvable.
- Selected targets are executable, or trustworthy prior execution evidence is available for them.
- Schema validator creation does not proceed until schema/service-definition source is explicitly provided or explicitly confirmed.

## 6) Procedure
### 6.1 Intake Classification
1. Confirm the request is direct validation-enrichment intent, for example:
   - add validations to existing tests,
   - enrich generated happy-path tests,
   - add schema validation,
   - compare response output with the DB,
   - add assertions to a DB Tool resultset.
2. If the request is still broad service-test creation/generation intent, route back through Skill 033 instead of continuing here.
3. Resolve target scope by precedence:
   - explicit values in the current prompt,
   - active project record sections relevant to the branch (`environment_files`, `services`, `facts`, `references`, `notes`),
   - values confirmed earlier in the current session,
   - relevant environment variables or existing asset references that have already been confirmed.
4. Ask scope once when unresolved:
   - which tests or DB Tool outputs should be enriched?
5. Ask follow-up questions only for unresolved enrichment-specific intent, one at a time:
   - should I propose the checks from the live responses/results, or do you want to specify them yourself?
   - should this include response-vs-database consistency checks where feasible?

### 6.1.1 API Capability Preflight (Required)
6. Before any write operation, run branch-aware capability preflight via `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.
7. Use the matched Skill 050 profile for the active branch, reuse/classify outcomes through Skill 050, and do not continue with optimistic endpoint assumptions when Skill 050 selects a fallback route.

### 6.1.2 Target Resolution Endpoint Selection (Required)
8. Resolve runtime targets using the API-first target-resolution policy from `docs/workflow/agent-workflow.md` plus the endpoint-selection mechanics in `docs/skills/platform/skill-001-shared-introspection.md`.
9. Keep this card's local note limited to enrichment-specific scope and handoff rather than duplicating the full endpoint-selection matrix.

### 6.2 Live Runtime Evidence Gate (Required)
10. Before proposing or attaching validation tooling, determine whether each selected target already has trustworthy runtime evidence available for inspection.
11. Trustworthy baseline evidence must come from the selected target's own execution traffic/results context:
  - prefer the selected target's execution traffic payloads and related run evidence,
  - do not substitute ad-hoc direct endpoint calls for baseline design.
  - the exploration-backed exception to this stable rule is owned only by experimental Skill 067 inside the explicit opt-in experimental lane.
12. If usable baseline evidence is missing, stale, or low-confidence:
  - run focused execution for the selected targets using Skill 012,
  - gather response traffic/results from that execution before proposing validation bundles.
13. For larger scopes, phase or batch the baseline runs instead of forcing one monolithic execution when phased collection is operationally safer or clearer.
14. Classify each selected target from runtime evidence:
  - producer family (`REST Client`, `SOAP Client`, `DB Tool`),
  - happy-path vs negative where applicable,
  - observed response/result media type and body/result shape,
  - obvious readiness-relevant configuration symptoms from the single observed output.
  - do not treat pre-remediation output characteristics as validation-bundle design input for any slice that is later classified as underconfigured.
### 6.2.1 Readiness Gate (Required)
15. Before proposing validation bundles, determine whether selected happy-path client tests or DB Tool validation targets appear meaningfully configured and runtime-ready.
16. Treat request-readiness remediation as a fail-closed prerequisite for validation enrichment when combined evidence indicates the selected targets are not yet meaningfully configured.
17. Require either:
  - at least two independent underconfiguration signals, or
  - one strong configuration signal plus one matching runtime symptom,
  before classifying a target as underconfigured.
18. High-confidence underconfiguration signals may include, depending on producer family:
  - REST/SOAP configuration-shape signals:
    - empty or obviously default request payloads on supposed happy-path `POST`/`PUT`/`PATCH` tests,
    - unresolved placeholders,
    - missing required request fields implied by known request shape.
  - DB Tool configuration-shape signals:
    - blank or scaffold-like JDBC/query configuration,
    - missing SQL query text,
    - unresolved SQL placeholders or obviously incomplete connection fields.
  - matching runtime symptoms:
    - responses, failures, or tool errors that strongly indicate incomplete input/configuration rather than purposeful negative testing.
19. Do not classify a target as underconfigured from one weak clue alone.
20. Failure alone is not enough, and default-like scalars such as `0`, empty strings, or empty arrays/objects count only when surrounding semantics show meaningful user values are expected.
21. Explicitly exclude targets that are clearly intended negatives or otherwise not remediation candidates, including:
  - tests clearly intended negatives based on naming, expected-code strategy, deliberate request mutation, or branch context,
  - `GET`/`DELETE` requests where an empty body is normal,
  - connectivity/auth/server failures with otherwise well-populated requests,
  - empty-but-valid DB resultsets or "no rows found" outcomes.
22. If some selected targets are ready and others are not:
  - summarize the ready slice and the underconfigured slice separately,
  - require an explicit user decision about whether to remediate the underconfigured slice first or continue only with the already-ready slice and defer the rest,
  - do not implicitly merge slices,
  - do not use pre-remediation observations from the underconfigured slice to justify validation design for that slice later.
23. If both happy-path and standard-negative targets are in scope and are already approved and ready, keep them in one combined validation-enrichment plan by default rather than serializing them into separate user-visible phases.
24. Split happy-path and standard-negative enrichment into separate phases only when a real readiness split, blocker, or explicit user instruction requires it.
25. If selected targets are not ready:
  - stop validation-bundle design for that slice immediately,
  - do not propose validator/assertor/diff tooling for that slice,
  - do not use pre-remediation response/result observations from that slice to choose Diff Tool vs Assertor,
  - offer request-readiness remediation,
  - hand off to `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`,
  - after remediation and verification succeed, refresh runtime evidence for the remediated slice and resume validation enrichment only from that fresh post-remediation ready-state evidence.
### 6.2.2 Remediation-to-Validation Approval Boundary (Required)
26. Approval to execute Skill 058 remediation is not approval to attach validation tooling.
27. After Skill 058 returns verified-ready slices, rebuild validation-bundle design from fresh post-remediation evidence and present a fresh validation plan summary before any validator/assertor/diff writes.
### 6.3 Happy-Path API Validation Bundle Proposal
28. Only after a slice is confirmed ready, inspect the current live output for validation-bundle design, including any static-vs-dynamic characteristics needed for Diff Tool vs Assertor selection.
  - classify payload type from the observed semantic response/result body first and use response headers such as `Content-Type` only as supporting evidence.
  - if headers and observed payload disagree, route from the observed payload/body rather than the header value.
  - if the observed REST Client response payload/body is empty or HTML, treat that target as a special no-tool exception: do not propose validator/assertor/diff chaining, and rely on the REST Client expected response code alone as the validation outcome.
29. For happy-path JSON/XML REST/SOAP responses, propose `schema validator + content validation` by default:
  - JSON response -> JSON Validator + (JSON Assertor or Diff Tool JSON mode),
  - XML response -> XML Validator + (XML Assertor or Diff Tool XML mode).
30. Do not ask the user to choose validator vs assertor vs diff as the first intake decision.
31. Resolve the content-validation side of the bundle from the observed response using a one-pass heuristic:
  - mostly static-looking response -> prefer Diff Tool,
  - response with obvious volatile fields (for example generated ids, timestamps, tokens, session artifacts) -> prefer targeted Assertor,
  - when uncertain, the agent may choose either Diff Tool or Assertor, but must not run repeated executions solely to prove volatility in v1.
32. For non-JSON/XML happy-path responses:
  - if the observed payload/body is HTML, treat it as the same no-tool exception and rely on the REST Client expected response code alone,
  - otherwise, do not attach JSON/XML validators by default,
  - route to Diff Tool mode that matches the observed runtime payload type.
33. Present the proposed content checks after inspecting the live response:
  - for small scopes, show concise per-test proposed checks,
  - for larger scopes, summarize the proposal by grouped pattern rather than exhaustively listing every suggested check.
33a. Before finalizing the validation proposal, consult active project facts/notes/references for stored business-critical identifiers, global validation rules, or environment-specific caveats that should influence what matters to validate. Use that project context to prioritize or include checks; do not invent new business assertions from vague notes.

### 6.3.1 DB Tool Resultset Validation Proposal
34. For direct DB Tool resultset enrichment, inspect the observed semantic result output and propose XML Assertor or Diff Tool XML mode by default.
35. Use the canonical semantic chain parent `Results as XML` from Skill 018 rather than diagnostic traffic outputs.
36. Do not force response-vs-database comparison just because a DB Tool is involved.
37. Do not route DB Tool resultsets into XML Validator; XML Validator remains API-client response-output scoped in this skill family.
38. If the user asks to validate DB Tool output itself, keep the DB-resultset branch on XML Assertor or Diff Tool XML mode rather than schema-validator attachment.
39. Present concise proposed assertions or diff scope after inspecting the live resultset.

### 6.4 Schema-Source Confirmation Policy
40. Schema/service-definition sources for validators must come from:
  - explicit user-provided WSDL/XSD/OpenAPI inputs, or
  - active project references already stored for the branch and presented back to the user for confirmation, or
  - clearly discoverable existing references that are presented back to the user for confirmation.
41. Do not invent schema sources.
42. If a likely environment-variable-backed definition source is discovered:
  - present both the variable form and the resolved value for confirmation before use,
  - for example `${OPENAPI}` and `http://localhost:8090/parabank/services/bank/openapi.yaml`.
43. If the user declines to provide or confirm a definition source:
  - offer content-validation-only enrichment, or
  - offer well-formedness-only validator behavior only when the user explicitly agrees to that fallback.
44. When schema validation is approved:
  - keep validator message/operation mapping aligned to service-definition templates,
  - do not substitute concretized runtime values into contract-mapping fields when the downstream validator card requires template paths.

### 6.5 Negative-Test Policy Overlay
45. Negative-test handling is a policy overlay, not the primary intake branch.
46. For selected standard negative tests:
  - do not attach JSON/XML validators,
  - choose content validation from the observed negative response media type only,
  - do not assume the negative response media type matches the happy-path media type,
  - if the observed REST Client response payload/body is empty or HTML, do not attach any chained tools and keep the calibrated/approved expected response code as the only validation,
  - prefer Diff Tool for simple static-looking error payloads,
  - prefer Assertor when targeted field checks are more appropriate.
47. For security branches created through the Penetration Testing Tool workflow:
  - do not attach validator/assertor/diff tools,
  - report the branch as intentionally handled by the Penetration Testing Tool only.
48. If a selected standard negative test still appears to use default client expected-response-code behavior, surface that mismatch and offer correction through the appropriate client/negative-authoring logic before or alongside validation enrichment.

### 6.6 Response-vs-Database Consistency Branch
49. If the approved plan includes DB-backed consistency checks:
  - confirm DB readiness and any known connection/query constraints,
  - identify correlation keys between response content and DB result data,
  - orchestrate the response-vs-DB comparison using Skill 015 plus databank/assertor/diff skills as appropriate.
50. Keep DB setup/precondition authoring out of scope for this card.

### 6.7 Orchestration Mapping
51. Map the confirmed enrichment plan to downstream skills:
  - request-readiness remediation -> Skill 058
  - JSON Validator -> Skill 029
  - XML Validator -> Skill 030
  - JSON Assertor -> Skill 010
  - XML Assertor -> Skill 016
  - Diff Tool -> Skill 031
  - JSON Data Bank -> Skill 028
  - XML Data Bank -> Skill 019
  - DB Tool lifecycle -> Skill 015
  - datasource introspection -> Skill 051
  - execution/results/traffic verification -> Skills 012 and 014
52. Use producer-output-only chaining:
  - discover outputs from the producer tool itself and select semantic response/results output,
  - never infer chain parent from sibling tool presence.

### 6.8 Confirmation and Execution
53. Build a human-readable enrichment plan summary:
  - selected targets,
  - baseline-evidence source,
  - request-readiness status,
  - API or DB-resultset bundle proposal,
  - any empty/HTML-body no-tool exceptions,
  - any negative-test exceptions,
  - schema-source status,
  - DB-resultset-validation scope and/or DB-comparison scope,
  - assumptions/unknowns requiring confirmation.
54. When both happy-path and standard-negative targets are in scope and ready, present them in one combined validation plan summary and request one approval by default rather than staging a second follow-up approval for negatives.
55. Require explicit user confirmation of this validation plan before validation writes, even when the same targets were already approved for remediation in Skill 058.
56. After approval, treat the approved per-target validation bundle as binding for execution.
  - preserve mixed-media selection per target; do not collapse a mixed JSON/XML/text scope into one validation family or one tool type.
  - do not replace an approved JSON/XML Assertor branch with Diff Tool without explicit fresh user confirmation.
  - do not omit an approved JSON/XML Validator silently; approved validator coverage is a required deliverable unless the user explicitly re-approves a downgrade.
  - do not reinterpret field-level approved assertion intent as full-body regression diffing without a fresh approval step.
57. Execute the combined approved enrichment pass in one stage when no readiness split or blocker requires serialization.
58. After approval, attach/configure validation tools in the approved sequence and verify readback/results.
59. If one approved validation tool fails to create or chain, log the failure, do not silently substitute another family, continue only with unaffected approved enrichment steps where safe, and report skipped/failed/blocked tools with reasons at the end.

## 7) Validation
- No validation-tool attachment occurs before explicit plan confirmation.
- No validation bundle is designed from contract-only assumptions when live runtime evidence for the selected targets is absent.
- Underconfigured client tests or DB Tool validation targets fail closed into request-readiness remediation rather than proceeding directly to validation-bundle attachment.
- Pre-remediation runtime evidence for an underconfigured slice is not reused to choose validator/assertor/diff tooling for that slice.
- Remediation approval from Skill 058 never authorizes validation writes by itself.
- Only verified-ready post-remediation slices return here, and validation design is rebuilt from fresh post-remediation evidence before a new confirmation gate.
- When happy-path and standard-negative targets are both approved and ready, Skill 057 prefers one combined validation plan and one approval by default rather than separate user-visible enrichment phases.
- Underconfiguration classification requires multi-signal evidence rather than one weak clue or failure alone.
- Empty-but-valid DB resultsets do not trigger request-readiness remediation by themselves.
- Empty or HTML REST Client response payloads/bodies are special no-tool exceptions: do not attach chained validators/assertors/diffs, and rely on the REST Client expected response code alone as the validation outcome.
- Happy-path JSON/XML tests default to `schema validator + content validation` unless the user explicitly prefers otherwise.
- Once the user approves a per-target validation plan, execution preserves that approved family per target unless the user explicitly re-approves a change.
- Mixed-media scopes remain per-target during execution; they are not normalized into one shared validation family across the batch.
- Approved JSON/XML validator coverage is not silently omitted or downgraded to content-only coverage.
- When response headers and observed payload/body disagree, tool-family selection follows the observed payload/body.
- Direct DB Tool resultset enrichment defaults to XML Assertor or Diff Tool XML mode from observed semantic result output and does not route into XML Validator.
- Negative tests do not receive JSON/XML validators by default.
- Schema/service-definition sources are never invented.
- Environment-variable-backed schema-source candidates are echoed using both variable form and resolved value before use.
- Response-vs-database consistency checks are explicitly mapped rather than left to ad hoc LLM composition.

## 8) Failure Modes
- Selected tests cannot be resolved or are mis-scoped.
- No trustworthy execution evidence is available and execution cannot be run.
- Underconfigured targets are mistaken for validation-ready happy-path tests.
- Intentionally negative tests are mistaken for underconfigured tests.
- Empty-but-valid DB resultsets are mistaken for underconfigured DB Tools.
- Connectivity/auth/server failures are mistaken for request-readiness failures.
- Validation bundle is proposed from guessed media type rather than observed runtime traffic.
- Pre-remediation response/result evidence is reused to design validation tooling after a Skill 058 handoff.
- Validation writes proceed because remediation approval was mistakenly treated as validation-plan approval.
- Happy-path and standard-negative validation are serialized into separate user-visible phases even though no readiness split, blocker, or user instruction required the split.
- Chained validation tools are added even though the observed REST Client response payload/body is empty or HTML and the expected response code alone should have remained the validation.
- An approved per-target validation plan is silently substituted during execution.
- Approved JSON/XML validator coverage is omitted even though it was part of the confirmed plan.
- Response headers are trusted over the observed payload/body, causing validator/assertor family mismatch.
- Direct DB Tool resultset validation is misrouted into response-vs-database comparison when the user only asked to validate DB output itself.
- Direct DB Tool resultset validation is misrouted into XML Validator even though that validator workflow is API-client response-output scoped.
- Schema source is ambiguous or silently invented.
- Happy-path schema validation is silently downgraded without user approval.
- Negative tests inherit happy-path validation bundle by default.
- Response-vs-database comparison stalls on missing correlation keys or unclear query strategy.

## 9) Safety / Rollback
- Orchestration-first policy: avoid speculative writes before confirmation gate.
- Rollback is delegated to the downstream atomic skills used in the approved enrichment slices.

## 10) Reuse Notes
- This is a composite orchestration card; it does not replace atomic validator/assertor/diff/client cards.
- Use this card when the user's primary intent is to enrich existing/generated tests or DB Tool outputs with validation.
- If selected targets are not request-ready, this card may hand off to Skill 058 before continuing validation enrichment.
- Skill 033 may also hand off here as a follow-on enrichment phase after broad authoring/generation, readiness stabilization, and any approved negative calibration reach a stable state.
- Direct validation-enrichment prompts should route here from `docs/skills/skill-index.md` instead of forcing Skill 033 first.
- Preferred location for this class of cards: `docs/skills/composite-orchestration/`.

## 11) Prompt Sequence Template (Default)
Use this condensed sequence in order for prompts that still require clarification, asking one question at a time and waiting for each answer before continuing.
1. "Which tests or DB Tool outputs in the `.tst` should I enrich with validation?"
2. "Do you want me to propose the checks from the live responses/results, or do you want to specify the checks yourself?"
3. "Do you also want response-vs-database consistency checks where feasible?"
4. "If schema validation is desired, what WSDL/XSD/OpenAPI source should I use?"
5. "Proceed with this enrichment plan summary: <selected targets + baseline source + proposed bundles + schema source + DB scope>?"
