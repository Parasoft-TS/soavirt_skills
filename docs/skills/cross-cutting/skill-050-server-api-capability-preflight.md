# Skill 050: Server API Capability Preflight

## 1) Skill Name
Server API capability preflight (endpoint-method compatibility and fallback routing)

## 2) Objective
Detect server-specific endpoint/method compatibility before orchestration writes so workflows choose known-safe API paths on first attempt.

## 3) Scope
- In scope:
  - base URL reachability probe
  - endpoint-method compatibility probes for selected workflow branches
  - deterministic fallback routing when a method is unsupported (for example `405`)
  - recording run-local compatibility observations in `work/` artifacts
- Out of scope:
  - modifying OpenAPI cache files
  - replacing endpoint-level behavior in atomic skills

## 4) Inputs
- Required:
  - resolved `baseUrl`
  - intended workflow branch (for example create-empty-tst, execution-traffic, diff-tool-xml)
- Optional:
  - previously observed compatibility notes from current run artifacts

## 5) Preconditions
- Server is reachable on network.
- Authentication requirements are satisfied.

## 6) Procedure
1. Probe base API reachability with a lightweight read:
   - `GET /v6/children`
2. Build a branch-specific capability probe list before writes.
3. Probe each endpoint/method pair with the minimum safe call shape.
4. Classify results:
   - supported,
   - unsupported method (`405`),
   - unavailable/not found (`404`),
   - auth/license blocked (`401`/`403`).
5. Select documented fallbacks for unsupported paths.
6. Continue orchestration using only verified-supported paths.

## 7) Canonical Fallbacks (Current)
- `.tst` structure read after create/reuse:
  - preferred: `GET /v6/children?id=<tst-id>`
  - fallback context: `GET /v6/descendants/assets?id=<tst-id>`
  - do not assume `GET /v6/files/tsts?id=<tst-id>` is supported on every server.
- traffic diagnostics:
  - first call suite-level traffic: `GET /v6/testExecutions/{id}/traffic?entityId=<suite-id>`
  - then call returned viewer ids for focused payloads.

## 8) Validation
- Preflight output explicitly lists supported and fallback-selected paths.
- No write call uses an endpoint-method pair classified unsupported in the same run.

## 9) Failure Modes
- Preflight omitted, causing mid-run `405`/`404` retries.
- Over-broad probe list increases startup latency.
- Misinterpreting auth failures (`401`/`403`) as method incompatibility.

## 10) Reuse Notes
- Applies to SOAtest: Yes.
- Applies to Virtualize: Yes (same API compatibility risk pattern).
- Use as a dependency for composite orchestration and any atomic card that has known fallback routes.
