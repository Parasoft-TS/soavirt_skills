# Skill 074: Set Disabled State
## 1) Skill Name
Enable or disable one or more supported assets through the generic `/v6/tools/disabled` batch API.
## 2) Objective
Provide one atomic owner for ordinary enable/disable requests so agents stop burying disabled-state changes inside unrelated lifecycle cards or YAML edits.
## 3) Scope
- In scope:
  - set `disabled=true` or `disabled=false` for supported assets via `PUT /v6/tools/disabled`
  - batch or single-target operation using the same payload contract
  - readback confirmation for supported v1 asset kinds
- Supported v1 kinds:
  - tools
  - test suites
  - responder suites
- Out of scope:
  - rename, copy, move, delete, or broader configuration updates
  - direct test-node/readback handling until a later card codifies that branch
  - local YAML edits as a substitute for the generic disabled-state API
## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
## 4) Inputs
- Required:
  - one or more targets, each with:
    - asset kind
    - runtime id
    - intended `disabled` state
- Optional:
  - expected current disabled state for no-op detection
## 5) Preconditions
- Each target kind is supported by this card.
- Each target id resolves through a known readback endpoint.
- Capability preflight confirms `PUT /v6/tools/disabled`.
## 6) Procedure
1. Resolve and normalize the target list.
2. Fail closed if any target kind is outside the supported v1 set.
3. Read current state before mutation through the owning readback surface for each target kind:
   - tool -> `GET /v6/tools?id=<id>`
   - test suite -> `GET /v6/suites/testSuites?id=<id>`
   - responder suite -> `GET /v6/suites/responderSuites?id=<id>`
4. Build one batch payload:
   - `tools: [{ id, disabled }, ...]`
5. Submit `PUT /v6/tools/disabled`.
6. Read back each target through its owning readback surface.
7. Confirm postconditions for every target:
   - the asset is still readable
   - the readback `disabled` field matches the requested state
8. If command output is ambiguous, reconcile through readback before any retry.
## 7) Validation
- Disabled-state writes use the generic batch endpoint.
- Each supported asset kind is verified through its owning readback endpoint.
- Batch success is not assumed until every target's readback state matches the request.
## 8) Failure Modes
- Treating disabled-state as tool-only when the same contract also applies to suites.
- Updating `disabled` indirectly through broader PUT payloads when the dedicated batch endpoint exists.
- Skipping per-target readback after a batch write.
- Expanding this card to unsupported asset kinds without codified readback ownership.
## 9) Safety / Rollback
- Read-only by default? No
- Rollback approach:
  - preserve the pre-change disabled state for every target
  - if a target lands in the wrong state and remains readable, restore with a second batch call using the preserved states
## 10) Reuse Notes
- Use this card for pure enable/disable intent across supported asset kinds.
- Broader lifecycle cards may still reference disabled state as one editable field, but narrow enable/disable requests should route here first.
