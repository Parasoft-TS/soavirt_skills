# Skill 031: Diff Tool Modes Workflow (XML/JSON/Text/Binary)

## 1) Skill Name
Add and validate Diff Tool chained to live semantic runtime output (multi-mode)

## 2) Objective
Create a reusable workflow for chaining a Diff Tool to live semantic runtime output and comparing observed output against expected content, including mode-specific ignored-difference configuration.

## 3) Scope
- In scope:
  - create Diff Tool via `POST /v6/tools/diffTools`
  - configure Diff Tool via `PUT /v6/tools/diffTools?id=...`
  - readback via `GET /v6/tools/diffTools?id=...`
  - run focused execution and validate runtime behavior by following the Skill 012 execution-triad contract
  - validate and document mode-specific ignore settings for:
    - XML mode
    - JSON mode
    - Text mode
    - Binary mode
- Out of scope:
  - domain-specific expected-payload authoring strategies
  - auto-remediation of runtime-output mismatches

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- When called from Skill 067, this card may rely on upstream exploration-backed mode selection, target parent selection, and candidate expected-content basis, while still owning local Diff Tool create/update/readback/verification mechanics.

## 4) Inputs
- Required:
  - target output-provider parent id (typically response output, but may also be another semantic output such as DB Tool `Results as XML`)
  - Diff Tool name
  - expected runtime output source/content
- Optional:
  - selected diff mode (`xml`, `json`, `text`, `binary`) when preselected by user
  - ignore-difference settings (mode-specific)
  - execution filter (`testNames`) for focused runtime checks

## 5) Preconditions
- API reachable and authenticated.
- Parent output channel provides stable semantic runtime output content.
- Parent id resolves to an output-provider location that accepts chained tools.
- In the stable lane, baseline execution evidence is required before mode selection, and observed runtime output type must drive the selected Diff Tool mode. When this card is called from Skill 067 in the experimental lane, upstream exploration-backed payload classification, approved Diff Tool family selection, target parent selection, and candidate expected-content basis may satisfy that mode-selection gate before attachment, but post-attachment focused verification remains required.

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
2. In the stable lane, run baseline execution first and capture observed semantic runtime output + type from the producer's canonical evidence source. When this card is called from Skill 067, accept upstream exploration-backed payload classification, approved Diff Tool family selection, target parent selection, and candidate expected-content basis as authoritative for mode-selection purposes instead of rerunning baseline work only to rediscover them.
  - for API-client response outputs, baseline source must be the executed test's response traffic payload (`/testExecutions/{id}/traffic` -> selected `Traffic Viewer`) when stable-lane baseline discovery is still needed.
  - for DB Tool outputs, baseline source must be the canonical semantic result output from Skill 018 (for example `Results as XML`), not diagnostic traffic.
  - do not seed expected values from ad-hoc external endpoint calls or non-canonical diagnostics because they may not match the configured parent tool's actual runtime output.
  - classify payload type from the observed semantic payload/body first; use response headers such as `Content-Type` only as supporting evidence.
  - if response headers and observed payload disagree, select diff mode from the observed payload/body rather than the header value.
3. Select diff mode from observed or upstream-authoritative runtime output type:
  - this gate is fail-closed: do not infer mode from producer class or apply a user-preferred mode when runtime evidence or upstream-authoritative classification shows a different payload type.
  - XML output -> `xml`
  - JSON output -> `json`
  - plain text output -> `text`
  - binary payload -> `binary`
  - For JSON outputs, prefer Diff Tool only when the approved intent is stable whole-payload equality and the payload has few volatile fields. If many fields are dynamic (timestamps, generated ids, session tokens), or if the expected semantic content is inferred from earlier business responses or mutable shared/business state likely to drift across runs, prefer JSON Assertor (Skill 010) or caller-owned cross-step semantic checks instead.
  - For XML outputs, apply the same rule between Diff Tool XML mode and XML Assertor (Skill 016).
  - if the caller/orchestration has already approved JSON/XML Assertor or Validator coverage for this target, do not use Diff Tool as a silent substitute; use Diff Tool only when it is the approved family for that target or when plain-text fallback applies.
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
  - set each tool's `diffMode.type` from that tool's own observed runtime output type; do not bulk-apply one mode to multiple tools.
  - immediately read back the tool and verify persisted `diffMode.type` matches the intended mode for that target output.
  - for plain-text outputs, default expected value source to the baseline live output captured in step 2, then refine only with explicit user intent.
7. Read back tool and capture exact persisted config shape.
8. Execute focused verification run by following the Skill 012 execution-triad contract and inspect mismatch/pass behavior.
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
- Do not transpose the XML `literal.editor.text` wrapper into JSON or text mode; each mode's `regressionControl` envelope in Section 6.2 is intentionally different and must be authored mode-by-mode.
- After PUT, compare the persisted `toolSettings.diffMode` subtree against the intended mode-specific envelope before concluding configuration succeeded.
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
- When response headers and observed payload disagree, readback/configuration and runtime verification reflect the observed payload/body rather than the header value.
- Baseline expected content origin is traceable to the same test execution traffic payload used for media-type detection.
- Per-tool readback confirms no unintended mode overwrite across sibling Diff Tools.
- Post-config verification run is required before concluding skill success.

## 8) Failure Modes
- `400` invalid payload shape or unsupported mode/config fields.
- `404` invalid tool id or parent id.
- False failures when mode does not match actual runtime output type.
- Mode-selection drift when mode is chosen from producer class or user preference instead of observed runtime payload type.
- False failures when expected baseline is sourced outside the canonical runtime evidence for the target output (for example external call with different negotiation headers or diagnostic-only output).
- False negatives/positives when ignore-difference settings are misconfigured.
- Cross-mode portability risk: ignore settings valid in one mode may not map directly to another mode.
- Multi-endpoint overwrite risk: a bulk update can unintentionally set all Diff Tools to one mode (for example `text`).
- Over-broad ignore risk: ignoring unstable fields without user confirmation can mask real regressions.
- Skipping baseline capture for text mode can leave expected-value seed empty or stale.
- Approved JSON/XML Assertor or Validator coverage can be silently replaced by Diff Tool if this card is used outside its intended target-specific fallback boundary.
- Experimental-lane drift risk: this card ignores Skill 067's upstream exploration-backed mode selection, target parent, or candidate expected-content basis and redoes mode-selection work from scratch.

## 8.1) Plain-Text Fallback Rule
- When validation intent is requested but observed runtime output is plain text:
  1. do not attach JSON/XML assertor/validator by default,
  2. create Diff Tool in `text` mode for that specific plain-text target only,
  3. seed expected text from baseline live output,
  4. run verification and present concise diff summary before any ignore changes.

## 9) Safety / Rollback
- Write skill (adds/modifies chained tool nodes).
- Rollback:
  - `DELETE /v6/tools?id=<diff-id>`, or
  - restore prior config from saved GET snapshot.

## 10) Reuse Notes
- Primary target: SOAtest.
- Technically applicable in Virtualize, but not often used.
- Cross-cutting dependencies:
  - `docs/skills/cross-cutting/skill-017-output-chaining-model.md`
  - `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`
  - `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md` (JSON mode selector semantics when selector-like ignore paths are used)
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- When called from Skill 067, this card may rely on upstream exploration-backed mode selection, target parent selection, and candidate expected-content basis, while still owning local Diff Tool create/update/readback/verification mechanics.

### User Interaction Rule (Diff Failures)
- On diff failure, provide a concise human-readable summary before any config mutation:
  - where the difference occurred (field/property/XPath)
  - expected value vs actual value
  - whether difference appears volatile across runs
- Ask user whether to add ignored differences for those specific fields.
- Do not auto-apply ignores unless user explicitly confirms.
- Do not use diff failure as justification to replace an approved JSON/XML Assertor or Validator branch without a new approval step.

## 11) Optional Mode-Matrix Evidence Capture
When validating this starter card, capture one example per mode with:
1. create/update/readback payloads
2. focused execution result
3. extracted note of how ignore-difference settings are represented for that mode
