# Skill 018: Tool Output Map Cheat-Sheet (Living Reference)

## 1) Skill Name
SOAtest tool output map and chaining defaults

## 2) Objective
Maintain a compact, high-signal lookup for choosing the correct output-provider parent when chaining tools.

## 3) Usage Rule
- Consult this map before creating/moving any chained tool.
- Treat this document as the canonical registry for producer/output-provider parent mapping.
- Downstream tool-workflow skills must reference this map instead of embedding exhaustive local mapping lists.
- If a run reveals mismatch or new behavior, update this map in the same session.

## 4) Output Map

| Producer Tool | Output Channel | Output Class | Default Chaining Decision | Parent ID Pattern | Typical Consumer Families | Notes |
|---|---|---|---|---|---|---|
| REST Client | Response Traffic (runtime media type: JSON/XML) | Semantic payload | Preferred default | `<rest-client-id>/Response Traffic` | JSON/XML assertors, validators, data banks, Diff Tool | Primary business-validation channel for API responses. |
| REST Client | Response Traffic (runtime media type: plain text) | Semantic payload | Conditional (Diff text mode) | `<rest-client-id>/Response Traffic` | Diff Tool (text mode) | Do not attach JSON/XML tools unless runtime media type confirms JSON/XML. |
| REST Client | Response Transport Header | Header metadata payload | Conditional (header-focused chaining) | `<rest-client-id>/Response Transport Header` | Header Data Bank | Use when extraction/validation intent targets response headers rather than body content. |
| REST Client | Traffic Object | Diagnostic payload | Avoid by default | Producer-specific; map before use | Traffic Viewer, diagnostics/export tooling | Not default for business-validation chaining. |
| SOAP Client | Response SOAP Envelope | Semantic payload | Preferred default | `<soap-client-id>/Response SOAP Envelope` | XML Assertor, XML Data Bank, XML Validator, Diff Tool (XML mode) | Validated on HTTP-based and WSDL-originated-compatible SOAP Client examples; use XML tools against the response envelope rather than traffic diagnostics. |
| SOAP Client | Traffic Object | Diagnostic payload | Avoid by default | `<soap-client-id>/Traffic Object` | Traffic Viewer, diagnostics/export tooling | Not default for business-validation chaining. |
| DB Tool | Results as XML | Semantic payload | Preferred default | `<db-tool-id>/Results as XML` | XML Assertor, XML Data Bank, Diff Tool (XML mode) | Default business payload channel for DB resultsets. |
| DB Tool | Traffic Object | Diagnostic payload | Avoid by default | Producer-specific; map before use | Traffic Viewer, diagnostics/export tooling | Useful for inspection, not default for business-validation chaining. |

## 5) Output Provider Path Construction (Required)
Output providers that have no children are NOT discoverable via `/children` or `/descendants/assets`.
Resolve `parent.id` from Section 4 first, then construct the path from that row's `Parent ID Pattern`.

Construction flow:
1. Identify producer tool type and intended output class (semantic vs diagnostic).
2. Select the matching row in Section 4.
3. Instantiate the row's `Parent ID Pattern` with the concrete producer id.
4. Use the resulting value as `parent.id`.
5. If no matching row exists, fail closed and update Section 4 before chaining.

Current validated examples:
- REST Client semantic response output: `<rest-client-id>/Response Traffic`
- REST Client response header output: `<rest-client-id>/Response Transport Header`
- SOAP Client semantic response output: `<soap-client-id>/Response SOAP Envelope`
- DB Tool semantic results output: `<db-tool-id>/Results as XML`

These examples are not exclusive; new producer/output combinations must be added to Section 4 before use.

Resolved `parent.id` values are valid for chained-tool creation endpoints such as `POST /v6/tools/jsonAssertors`, `POST /v6/tools/jsonValidators`, `POST /v6/tools/jsonDataBanks`, and XML equivalents when media type and tool family are compatible.

The tool creation endpoints require `parent.id` to resolve to type `outputProvider` or `suite`. Passing a REST Client id directly (type `REST Client`) will return `400`.

### 5.1) Traffic Retrieval Normalization (Required)
- For runtime request/response diagnostics, use this sequence:
  1. `GET /v6/testExecutions/{id}/traffic?entityId=<suite-id>`
  2. select returned `trafficViewers[].id`
  3. `GET /v6/testExecutions/{id}/traffic?entityId=<traffic-viewer-id>`
- Do not assume direct traffic retrieval from a guessed output-provider path will return populated runs.

## 6) Selection Algorithm
1. Identify the producer tool and resolve its output-provider mapping from this card first.
  - if no mapping exists for the producer/output pair, fail closed: stop chaining and update this card before continuing.
2. Construct the output-provider path using the mapped pattern in Section 5.
  - do NOT rely on `/children` or `/descendants/assets` to discover output providers; they may be empty and invisible.
  - use producer tool/output discovery only; ignore sibling-tool heuristics.
3. Match required payload type (JSON/XML/text) to semantic output channel.
  - enforce media-type gate from runtime evidence before selecting tool family.
  - JSON tools only for JSON payloads; XML tools only for XML payloads; plain text -> Diff Tool text mode.
4. Choose semantic output (response/results) before diagnostics channels.
5. Create chain and run focused execution.
6. If parser/type mismatch occurs, re-evaluate parent selection before changing assertions.

### 6.1) Hard Guard
- Do not use `Traffic Viewer` presence as chain-parent evidence.
- `Traffic Viewer` is a common sibling artifact and is frequently misleading for business-validation chaining.

### 6.2) Request-Oriented Outputs (Rare Exception)
- Treat request-oriented outputs as non-default chaining targets.
- Use them only when user intent explicitly targets request artifacts (for example Write File export or Extension Tool processing).
- Require explicit intent + validation evidence before attaching tools to request-side outputs.

## 7) Maintenance Protocol (Required)
- When new tool/output behavior is validated:
  1. Add/update a row in this map.
  2. Add a short evidence note in `docs/logs/chat-log.md`.
  3. If policy changed, update `docs/skills/cross-cutting/skill-017-output-chaining-model.md`.
- Keep rows concise and behavior-focused (purpose + default chaining decision).
- Do not proceed with unmapped producer/output chaining in workflow skills until this map is updated.

### 7.1) Row Authoring Template (for New Entries)
When adding a new producer/output mapping row, always include:
- `Producer Tool`: concrete tool family name.
- `Output Channel`: exact channel label as exposed in the tool tree.
- `Output Class`: `Semantic payload` or `Diagnostic payload`.
- `Default Chaining Decision`: preferred/conditional/avoid-by-default.
- `Parent ID Pattern`: deterministic pattern for `parent.id` construction.
- `Typical Consumer Families`: tool families expected to chain here.
- `Notes`: media-type gates, caveats, or constraints.

## 8) Known Anti-Pattern
- Chaining business assertors to `Traffic Object` by default.
  - Typical symptom: parser/bootstrap errors (for example XML prolog/syntax mismatch) instead of meaningful business assertion evaluation.

## 9) Example Mapping Correction
- DB Tool correction example:
  - from `.../Traffic Object/...` (mismatch)
  - to `.../Results as XML/...` (pass)
