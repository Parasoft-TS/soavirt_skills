# Skills Taxonomy Refactor Proposal (2026-03-04)

## 1) Why Refactor the Taxonomy
- Current organization is mostly lifecycle/layer-oriented (L0-L4), which is strong for dependency sequencing.
- Day-to-day authoring and troubleshooting questions are often intent-oriented (for example: "I need a client tool", "I need data extraction", "I need validation").
- A dual-view taxonomy (Layer + Intent Domain) improves discoverability without breaking current skill IDs.

## 2) Proposed Intent Domains

### A. Platform & Asset Lifecycle
Purpose: server/workspace file operations and asset creation.

Includes:
- `skill-001` shared introspection
- `skill-002` shared file transfer
- `skill-003` copy/rename
- `skill-004` rename in place
- `skill-005` delete
- `skill-006` safe refactor composite
- `skill-021` create empty `.tst`
- `skill-022` create `.tst` from OpenAPI/Swagger
- `skill-023` create `.tst` from WSDL
- `skill-024` create `.tst` from RAML
- `skill-025` create `.tst` from XSD

### B. Structural Authoring (TST Object Manipulation)
Purpose: place/move/restructure suites, tools, and data sources.

Includes:
- `skill-family-tst-object-manipulation`
- `skill-008` datasource type-targeted move
- `skill-009` testsuite create/configure
- planned atomic cards: reorder, move, copy, rename, delete overlays

### C. Client Tools
Purpose: configure tools that call external systems and produce primary runtime payloads.

Includes:
- `skill-020` REST Client workflow (None mode)
- `skill-015` DB Tool lifecycle
- deferred constrained REST Client path (former 026/027 concept)

### D. Data Exchange Tools
Purpose: extract and propagate values from runtime outputs for downstream use.

Includes:
- `skill-019` XML Data Bank extraction workflow
- `skill-028` JSON Data Bank extraction workflow

### E. Validation Tools
Purpose: assert and validate runtime content/behavior.

Includes:
- `skill-010` JSON Assertor CRUD/copy
- `skill-016` XML Assertor workflow
- `skill-029` JSON Validator workflow
- `skill-030` XML Validator workflow
- `skill-031` Diff Tool modes workflow (XML/JSON/Text/Binary)

### F. Execution & Diagnostics
Purpose: run tests, capture evidence, explain failures.

Includes:
- `skill-012` test execution + XML report
- `skill-014` failure diagnostics from traffic

### G. Cross-Cutting Policies & Semantics
Purpose: normalize behavior across all domains.

Includes:
- `skill-007` TST summarization/intent mapping
- `skill-011` XPath-over-JSON semantics
- `skill-013` deterministic naming policy
- `skill-017` output chaining model
- `skill-018` output map cheat-sheet

## 3) Clarification on Grouping Examples
- Data Banks should be grouped under **Data Exchange Tools** (agreed).
- Assertors should be grouped under **Validation Tools**.
  - If a prior note said "data exchange" for assertors, treat that as a wording slip; semantically they belong to validation.
- REST Client and DB Tool belong under **Client Tools** (agreed).

## 4) Recommended Information Architecture (Current State)
- Keep numeric skill IDs stable for continuity.
- Intent-domain section is now present in `skill-index.md` as a second navigation surface.
- Skill files are now organized into shallow domain folders under `docs/skills/`.
- Keep `backlog.md` endpoint-focused, and tag each new backlog item with an intent domain over time.
- Add one short `Domain:` field to each skill card as the next normalization step.

## 5) Proposed Future Folder Layout (Optional, Phase 2+)
If you later choose physical reorganization, use folders while preserving numeric IDs:

- `docs/skills/platform/`
- `docs/skills/structure/`
- `docs/skills/client-tools/`
- `docs/skills/data-exchange/`
- `docs/skills/validation/`
- `docs/skills/execution-diagnostics/`
- `docs/skills/cross-cutting/`

Naming example:
- `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`

## 6) Phased Refactor Plan
1. **Phase 1 (completed):** Add intent-domain index view and taxonomy proposal doc.
2. **Phase 2 (completed):** Move skill files into shallow domain folders and update references.
3. **Phase 3 (next):** Add `Domain:` metadata line to each existing skill card.
4. **Phase 4 (next):** Fill gaps with dedicated cards:
  - run-time validation pass for JSON Validator card
  - run-time validation pass for XML Validator card
  - per-mode configuration/evidence pass for Diff Tool (XML/JSON/Text/Binary)
5. **Phase 5 (optional):** Add role-based index shortcuts (author/build/diagnose) if navigation still feels heavy.

## 7) Acceptance Criteria for Taxonomy Refactor
- A new contributor can answer "which skills do I load for this intent?" in under 30 seconds.
- Every skill appears in exactly one primary intent domain (plus optional cross-cutting references).
- Existing workflows and links remain valid after folder migration.
- Validation/diagnostics guidance remains consistent with `skill-014` semantics for status-code interpretation and chained-tool cascade analysis.
