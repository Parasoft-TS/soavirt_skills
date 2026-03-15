# Skill Family: Disabled-State Mutation
Purpose: route enable/disable requests to the broader cross-asset disabled-state owner instead of treating them as tool-only lifecycle edits.
## Scope
This family covers supported disabled-state changes that share the same `/v6/tools/disabled` batch payload contract.
Supported v1 asset kinds:
- tools
- test suites
- responder suites
The endpoint description also mentions tests/responders more broadly, but direct readback ownership for those narrower kinds is not yet codified in this family.
## Foundational Skills
- `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- `docs/skills/structure/skill-074-set-disabled-state.md`
## Selection Guide
- Need to enable or disable one supported asset without broader configuration changes?
  - use `docs/skills/structure/skill-074-set-disabled-state.md`
- Need to rename an object?
  - use `docs/skills/structure/skill-068-rename-object.md`
- Need to copy, move, or delete a tool?
  - use `docs/skills/structure/skill-family-tool-lifecycle-operations.md`
## Boundary Rules
- Treat disabled-state changes as a cross-asset state primitive, not as a tool-only lifecycle detail.
- Do not use local YAML editing for ordinary disabled-state changes when the generic disabled-state endpoint is available.
