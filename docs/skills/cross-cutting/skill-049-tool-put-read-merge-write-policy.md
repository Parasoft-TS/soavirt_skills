# Skill 049: Tool PUT Read-Merge-Write Policy

## 1) Skill Name
Safe update policy for existing tool configurations using GET -> mutate -> PUT.

## 2) Objective
Prevent accidental configuration loss when editing pre-configured tools by requiring read-merge-write updates instead of sparse PUT payloads.

## 3) Scope
- In scope:
  - updating existing SOAtest/Virtualize tools via `PUT`
  - preserving current configuration fields not intentionally changed
  - pre-update capture and post-update verification checks
- Out of scope:
  - creating new tools from scratch (`POST` workflows)
  - endpoint-specific create payload design
  - non-tool object update semantics (`files`, `suites` ordering, etc.)

## 4) Inputs
- Required:
  - target tool id
  - intended field-level changes
- Optional:
  - allowlist of fields expected to change
  - rollback snapshot location

## 5) Preconditions
- API reachable and authenticated.
- Target tool exists and is readable by `GET`.
- Update target is an existing (already configured) tool.

## 6) Procedure
1. Read current tool config:
   - `GET` class-specific endpoint for the tool (for example `/v6/tools/restClients?id=...`, `/v6/tools/dbTools?id=...`, `/v6/tools/jsonValidators?id=...`).
2. Build PUT payload by merge:
   - start from current config fields accepted by that endpoint's PUT schema,
   - apply only intended mutations,
   - preserve all other fields as-is.
3. Execute update:
   - `PUT` class-specific endpoint with merged payload.
4. Verify readback:
   - `GET` same tool id,
   - confirm changed fields match intent,
   - confirm critical unchanged fields remain intact.
5. If verification fails:
   - stop downstream actions,
   - restore from pre-update snapshot if available.

## 7) Validation
- Required checks:
  - intended mutations are present after PUT,
  - unrelated settings did not regress,
  - critical execution fields remain non-empty (for example tool target URL/path/message mapping where applicable).

## 8) Failure Modes
- Sparse PUT overwrite: sending only a subset of fields clears pre-existing configuration.
- Endpoint schema drift: field names/structure mismatch across tool types.
- Post-update silent no-op execution due to wiped required fields.

## 9) Safety / Rollback
- Treat every tool PUT as destructive-risk unless proven otherwise.
- Keep a pre-update snapshot (`GET` response) for fast rollback.

## 10) Reuse Notes
- Applies to SOAtest: Yes (validated pattern).
- Applies to Virtualize: Yes (policy-level; endpoint-specific validation pending).
- Composition rule:
  - load this card before any skill that edits existing tool configurations.
  - `POST` create flows are exempt unless they transition into editing an existing tool instance.

## 11) Current Validation Status (2026-03-04)
- Validated in current workspace for REST Client updates:
  - name-only sparse `PUT /v6/tools/restClients` cleared URL/resource settings.
  - read-merge-write PUT preserved required URL fields while applying intended mutations.
- Cross-tool policy promoted from this observed behavior to avoid similar regressions in other pre-configured tool types.
