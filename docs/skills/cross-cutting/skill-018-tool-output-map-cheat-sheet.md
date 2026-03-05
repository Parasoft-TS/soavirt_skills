# Skill 018: Tool Output Map Cheat-Sheet (Living Reference)

## 1) Skill Name
SOAtest tool output map and chaining defaults

## 2) Objective
Maintain a compact, high-signal lookup for choosing the correct output-provider parent when chaining tools.

## 3) Usage Rule
- Consult this map before creating/moving any chained tool.
- If a run reveals mismatch or new behavior, update this map in the same session.

## 4) Output Map (Current Validated Knowledge)

| Producer Tool | Output Channel | Default Purpose | Chain Assertor/Validator Here? | Notes |
|---|---|---|---|---|
| REST Client | Response Traffic / response-oriented output | Business payload validation | Yes (default) | Preferred parent for JSON/XML assertors, data banks, and validators. |
| REST Client | Response Traffic / plain-text response | Textual response validation | Use Diff Tool (text mode) | Do not attach JSON/XML tools unless runtime media type confirms JSON/XML. |
| REST Client | Traffic Object | Runtime diagnostics visualization | Usually no | Primarily for `Traffic Viewer`; use only for explicit diagnostics/export use cases. |
| DB Tool | Results as XML | SQL result payload for assertions | Yes (default) | Correct parent for XML Assertor over DB resultsets. |
| DB Tool | Traffic Object | JDBC request/response diagnostics | Usually no | Useful for traffic inspection; not default for business assertions. |
| DB Tool | Results as XML | SQL result payload extraction | Yes (default) | Correct parent for XML Data Bank XPath extraction (for example ID/BALANCE). |

## 5) Output Provider Path Construction (Required)
Output providers that have no children are NOT discoverable via `/children` or `/descendants/assets`. Construct the `parent.id` path directly using the patterns below.

- REST Client response output: `<rest-client-id>/Response Traffic`
- DB Tool results output: `<db-tool-id>/Results as XML`

These paths are valid `parent.id` values for `POST /v6/tools/jsonAssertors`, `POST /v6/tools/jsonValidators`, `POST /v6/tools/jsonDataBanks`, and their XML equivalents.

The tool creation endpoints require `parent.id` to resolve to type `outputProvider` or `suite`. Passing a REST Client id directly (type `REST Client`) will return `400`.

### 5.1) Traffic Retrieval Normalization (Required)
- For runtime request/response diagnostics, use this sequence:
  1. `GET /v6/testExecutions/{id}/traffic?entityId=<suite-id>`
  2. select returned `trafficViewers[].id`
  3. `GET /v6/testExecutions/{id}/traffic?entityId=<traffic-viewer-id>`
- Do not assume direct traffic retrieval from a guessed output-provider path will return populated runs.

## 6) Selection Algorithm
1. Identify the producer tool and construct the output-provider path using the patterns in Section 5.
  - do NOT rely on `/children` or `/descendants/assets` to discover output providers; they may be empty and invisible.
  - use producer tool/output discovery only; ignore sibling-tool heuristics.
2. Match required payload type (JSON/XML/text) to semantic output channel.
  - enforce media-type gate from runtime evidence before selecting tool family.
  - JSON tools only for JSON payloads; XML tools only for XML payloads; plain text -> Diff Tool text mode.
3. Choose semantic output (response/results) before diagnostics channels.
4. Create chain and run focused execution.
5. If parser/type mismatch occurs, re-evaluate parent selection before changing assertions.

### 6.1) Hard Guard
- Do not use `Traffic Viewer` presence as chain-parent evidence.
- `Traffic Viewer` is a common sibling artifact and is frequently misleading for business-validation chaining.

## 7) Maintenance Protocol (Required)
- When new tool/output behavior is validated:
  1. Add/update a row in this map.
  2. Add a short evidence note in `docs/logs/chat-log.md`.
  3. If policy changed, update `docs/skills/cross-cutting/skill-017-output-chaining-model.md`.
- Keep rows concise and behavior-focused (purpose + default chaining decision).

## 8) Known Anti-Pattern
- Chaining business assertors to `Traffic Object` by default.
  - Typical symptom: parser/bootstrap errors (for example XML prolog/syntax mismatch) instead of meaningful business assertion evaluation.

## 9) Validation Snapshot (2026-03-03)
- DB Tool correction validated:
  - from `.../Traffic Object/...` (mismatch)
  - to `.../Results as XML/...` (pass)
