# Skill 031: Diff Tool Modes Workflow (XML/JSON/Text/Binary)

## 1) Skill Name
Add and validate Diff Tool chained to live response output (multi-mode)

## 2) Objective
Create a reusable workflow for chaining a Diff Tool to a live response output and comparing runtime output against expected content, including mode-specific ignored-difference configuration.

## 3) Scope
- In scope:
  - create Diff Tool via `POST /v6/tools/diffTools`
  - configure Diff Tool via `PUT /v6/tools/diffTools?id=...`
  - readback via `GET /v6/tools/diffTools?id=...`
  - run focused execution and validate runtime behavior via:
    - `POST /v6/testExecutions`
    - `GET /v6/testExecutions/{id}/status`
    - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
  - validate and document mode-specific ignore settings for:
    - XML mode
    - JSON mode
    - Text mode
    - Binary mode
- Out of scope:
  - domain-specific expected-payload authoring strategies
  - auto-remediation of response mismatches

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`

## 4) Inputs
- Required:
  - target output-provider parent id (typically response output)
  - Diff Tool name
  - expected response source/content
- Optional:
  - selected diff mode (`xml`, `json`, `text`, `binary`) when preselected by user
  - ignore-difference settings (mode-specific)
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel provides stable runtime response content.
- Parent id resolves to an output-provider location that accepts chained tools.
- Baseline execution evidence is required before mode selection, and observed runtime payload type must drive the selected Diff Tool mode.

## 6) Procedure
0. Apply capability preflight before first write:
  - use Skill 050 Profile E for tool mutation steps,
  - use Skill 050 Profile D for execution/traffic-observation validation steps.
1. Resolve the target producer/output-provider parent using the canonical registry:
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md` (Section 5).
  - Construct `parent.id` from Skill 018 mapping; do NOT pass the producer tool id directly.
1.1 Fail-closed guard:
  - if the producer/output pair is not mapped in Skill 018, stop and request a Skill 018 update before creating/configuring Diff Tool.
  - do not guess or locally invent parent-path mappings.
2. Run baseline execution first and capture observed response payload + content type from traffic evidence.
  - baseline source must be the executed test's response traffic payload (`/testExecutions/{id}/traffic` -> selected `Traffic Viewer`).
  - do not seed expected values from ad-hoc external endpoint calls because headers/content negotiation may differ from the configured parent tool's run.
3. Select diff mode from observed runtime payload type:
  - this gate is fail-closed: do not infer mode from producer class or apply a user-preferred mode when runtime evidence shows a different payload type.
  - XML response -> `xml`
  - JSON response -> `json`
  - plain text response -> `text`
  - binary payload -> `binary`
  - For JSON responses, Diff Tool is preferred when response data is mostly static (few volatile fields). If many response fields are dynamic (timestamps, generated ids, session tokens), consider JSON Assertor (Skill 010) instead — Diff Tool would require many ignored differences, reducing its value since those elements are not tested anyway.
  - For XML responses, apply the same static/dynamic heuristic between Diff Tool XML mode and XML Assertor (Skill 016).
4. Resolve Diff Tool identity before writes (idempotent upsert rule):
  - derive deterministic target id as `<parent-id>/<diff-name>` for the selected comparison intent,
  - if that id exists, update in place,
  - create via `POST /v6/tools/diffTools` only when Diff Tool is absent,
  - do not create another Diff Tool with the same intent/name in the same parent.
5. Create Diff Tool when absent:
   - `POST /v6/tools/diffTools` with `parent.id` and `name`.
6. Configure base comparison settings:
   - `PUT /v6/tools/diffTools?id=<diff-id>` with expected content/source and mode.
  - for updates to existing Diff Tools, follow read-merge-write (`GET` -> mutate -> `PUT`) per Skill 049.
  - set each tool's `diffMode.type` from that tool's own observed response media type; do not bulk-apply one mode to multiple tools.
  - immediately read back the tool and verify persisted `diffMode.type` matches the intended mode for that endpoint.
  - for plain-text responses, default expected value source to baseline live response captured in step 2, then refine only with explicit user intent.
7. Read back tool and capture exact persisted config shape.
8. Execute focused verification run and inspect mismatch/pass behavior.
9. If diff fails, extract and summarize human-readable differences (field/XPath/property, expected vs actual, source tool context).
10. Offer optional ignored-difference setup for reported volatile fields:
  - apply only after user confirmation.
  - keep strict comparison as default when user does not request ignores.
11. If ignores were added, run focused verification again to confirm expected pass/fail behavior.
12. Repeat steps 6-11 for each mode (`xml`, `json`, `text`, `binary`) and capture config + runtime evidence.

### 6.1) Payload-Shape Usage Guard (Disambiguation Only)
The payloads in Section 6.2 are shape references, not semantic defaults.

Rules:
- Replace all placeholders (`<diff-name>`, `<expected-xml>`, `<expected-json>`, `<expected-text>`) using runtime evidence + user intent.
- Do not copy expected-content literals from examples.
- Choose mode from baseline runtime media type (Section 6, steps 2-3), not from preference.
- For updates, use GET -> mutate -> PUT read-merge-write per Skill 049.
- Keep strict comparison by default; apply ignored differences only after explicit user confirmation.

### 6.2 Canonical PUT Payload Shapes (Mode-Safe)
Use these minimal shapes when configuring expected content to avoid schema-shape drift.

XML mode baseline:
```json
{
  "name": "<diff-name>",
  "toolSettings": {
    "diffMode": {
      "type": "xml",
      "xml": {
        "regressionControl": {
          "type": "literal",
          "literal": {
            "type": "editor",
            "editor": {
              "text": "<expected-xml>"
            }
          }
        }
      }
    }
  }
}
```

JSON mode baseline:
```json
{
  "name": "<diff-name>",
  "toolSettings": {
    "diffMode": {
      "type": "json",
      "json": {
        "regressionControl": {
          "type": "literal",
          "editor": {
            "text": "<expected-json>"
          }
        }
      }
    }
  }
}
```

Text mode baseline:
```json
{
  "name": "<diff-name>",
  "toolSettings": {
    "diffMode": {
      "type": "text",
      "text": {
        "regressionControl": {
          "type": "editor",
          "editor": {
            "text": "<expected-text>"
          }
        }
      }
    }
  }
}
```

## 7) Validation
- Create/update/readback calls return `200` on valid payloads.
- Runtime result indicates diff pass/fail aligned to expected content and ignore settings.
- Repeat executions/update passes do not create additional Diff Tools for the same parent/name intent.
- For each mode, persisted ignore settings are documented from real readback payloads.
- Baseline run evidence confirms mode selection used observed payload type (not assumption).
- Baseline expected content origin is traceable to the same test execution traffic payload used for media-type detection.
- Per-tool readback confirms no unintended mode overwrite across sibling Diff Tools.
- Post-config verification run is required before concluding skill success.

## 8) Failure Modes
- `400` invalid payload shape or unsupported mode/config fields.
- `404` invalid tool id or parent id.
- False failures when mode does not match actual response content type.
- Mode-selection drift when mode is chosen from producer class or user preference instead of observed runtime payload type.
- False failures when expected baseline is sourced outside runtime traffic (for example external call with different negotiation headers).
- False negatives/positives when ignore-difference settings are misconfigured.
- Cross-mode portability risk: ignore settings valid in one mode may not map directly to another mode.
- Multi-endpoint overwrite risk: a bulk update can unintentionally set all Diff Tools to one mode (for example `text`).
- Over-broad ignore risk: ignoring unstable fields without user confirmation can mask real regressions.
- Skipping baseline capture for text mode can leave expected-value seed empty or stale.

## 8.1) Plain-Text Fallback Rule
- When validation intent is requested but observed runtime response is plain text:
  1. do not attach JSON/XML assertor/validator by default,
  2. create Diff Tool in `text` mode,
  3. seed expected text from baseline live response,
  4. run verification and present concise diff summary before any ignore changes.

## 9) Safety / Rollback
- Write skill (adds/modifies chained tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<diff-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Virtualize applicability may differ by product object model and should be checked before reuse.
- Use `docs/skills/backlog.md` for current validation and coverage status.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md` (JSON mode selector semantics when selector-like ignore paths are used)
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md` (use GET -> mutate -> PUT for updates to existing Diff Tool configurations)
  - when new producer/output types are introduced, update Skill 018 first, then apply here.

### User Interaction Rule (Diff Failures)
- On diff failure, provide a concise human-readable summary before any config mutation:
  - where the difference occurred (field/property/XPath)
  - expected value vs actual value
  - whether difference appears volatile across runs
- Ask user whether to add ignored differences for those specific fields.
- Do not auto-apply ignores unless user explicitly confirms.

## 11) Optional Mode-Matrix Evidence Capture
When validating this starter card, capture one example per mode with:
1. create/update/readback payloads
2. focused execution result
3. extracted note of how ignore-difference settings are represented for that mode
