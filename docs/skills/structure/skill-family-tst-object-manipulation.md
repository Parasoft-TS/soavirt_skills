# Skill Family: TST Object Manipulation

Purpose: define a scalable, context-efficient skill architecture for copy/paste/move-style manipulation across `.tst` object types.

## Scope
This family covers structural operations on these object classes:
- test suites
- tools
- data sources
- environments
- global properties

## Design Rule
- Keep one umbrella family card for intent routing.
- Implement operation behavior in atomic cards.
- Use composite cards only for repeated user workflows.

## Atomic Skill Matrix

### A) Operation-Centric Core (required first)
1. `copy-object` (server-side where supported)
2. `move-object` (reparent to new parent)
3. `reorder-children` (order within same parent)
4. `rename-object` (in-place identity label change)
5. `delete-object` (remove node)

Each operation card must define:
- supported object classes
- exact endpoint(s)
- required ids and constraints
- preflight checks
- post-condition verification
- rollback guidance

### B) Object-Class Overlays (thin deltas)
For each object class, add a thin overlay section/card that states:
- endpoint deviations from core operation cards
- object-specific constraints (for example parent compatibility)
- additional verification probes

Object overlays:
- suite overlay
- tool overlay
- data source overlay
- environment overlay
  - canonical v1 owner: `docs/skills/structure/skill-064-soatest-environment-lifecycle.md`
- global property overlay

## Composite Workflow Cards
Use composites only for high-frequency intents:
1. copy + reparent + reorder ("paste into target and place at position")
2. move + normalize name ("move then rename to convention")
3. clone branch workflow ("copy suite subtree and verify descendants")

## Dependency Model
- L2 atomic operations depend on L0 introspection/download-upload safety and L1 structure mapping.
- L2 object overlays depend on the L2 operation core card they extend.
- L2 composites depend on multiple L2 atomic cards and should not duplicate endpoint logic.

## Validation Standard
Every atomic manipulation card must include:
1. Read-only preflight discovery
2. Explicit mutation request format
3. Before/after evidence capture
4. Failure mode mapping (constraint violation, missing id, parent mismatch)

## Known Constraints
- **Environment child ordering is constrained**:
	- `PUT /v6/suites/children` does not allow arbitrary `QA`/`DEV` swaps in this target file.
	- Even with explicit `DEV` then `QA` payload, persisted order remained `QA` then `DEV`.
	- Treat environment reordering as constrained/not supported unless a future product/API version proves otherwise.
- **Data source move can be implementation-ambiguous for shared display ids**:
	- `POST /v6/datasources/move` may move one underlying implementation per call when multiple data source implementations are represented under the same visible node id/path.
	- Validation must use before/after YAML (implementation type traces) in addition to REST child listings.
	- For requests like "move TableDataSource" / "move FileDataSource" / "move WritableDataSource", confirm target implementation type explicitly after move.

## Data Source Move Pattern (Generalized)
- This skill is **type-targeted**, not table-specific.
- Target any implementation type under `dataSources`, including:
	- `TableDataSource`
	- `FileDataSource`
	- `WritableDataSource`
	- `CSVDataSource`
	- `DbDataSource`
	- `ExcelDataSource`
- Preferred execution path:
	1. Create/resolve destination suite via REST.
	2. Detect if source has one-to-one id mapping or shared display id ambiguity.
	3. If ambiguous, perform surgical YAML move of the chosen implementation block.
	4. Upload and verify by implementation type trace in source and destination suites.

