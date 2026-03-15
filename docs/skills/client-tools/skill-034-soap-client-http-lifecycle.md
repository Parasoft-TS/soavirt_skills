# Skill 034: SOAP Client Lifecycle (HTTP-Focused)
## 1) Skill Name
SOAP Client lifecycle on API-exposed HTTP surface (create/read/update/delete/copy + optional reorder/validation)

## 2) Objective
Manage SOAtest SOAP Client tests through the API-exposed request/transport/misc surface, covering scaffold creation, configured HTTP request authoring, safe updates, and subordinate copy/delete steps when they are part of broader SOAP Client lifecycle work.
Pure rename/copy/delete/enable-disable requests on an existing SOAP Client should route to the centralized operation-centric owners in `docs/skills/structure/skill-068-rename-object.md`, `docs/skills/structure/skill-070-copy-tool.md`, `docs/skills/structure/skill-072-delete-tool.md`, and `docs/skills/structure/skill-074-set-disabled-state.md`.

First-pass boundary:
- validated for HTTP-based SOAP Clients,
- compatible with WSDL-originated SOAP Clients whose runnable behavior is preserved through copy and read-merge-write updates of exposed fields,
- does not claim full UI parity for WSDL/XML Schema resource-mode settings or WS-Policy configuration.

## 3) Scope
- In scope:
  - create SOAP Client via `POST /v6/tools/soapClients`
  - update SOAP Client via `PUT /v6/tools/soapClients?id=...`
  - read SOAP Client via `GET /v6/tools/soapClients?id=...`
  - delete SOAP Client via `DELETE /v6/tools?id=...`
  - copy SOAP Client via `POST /v6/tools/copy`
  - reorder tool placement under suite via `PUT /v6/suites/children?id=...`
  - focused runtime validation via `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Out of scope:
  - WS-Policy configuration
  - full UI-equivalent WSDL/XML Schema resource-mode authoring
  - non-HTTP transport authoring beyond current validated evidence
  - downstream XML Data Bank / XML Assertor authoring (owned by Skills 019 and 016 plus Skill 018 mapping)

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`
- Additive:
  - `docs/skills/cross-cutting/skill-013-test-naming-policy.md`
  - `docs/skills/cross-cutting/skill-032-client-header-ownership.md`

## 4) Inputs
- Required:
  - parent suite id
- Optional:
  - SOAP Client name (optional - derive from explicit operation name when unambiguous; otherwise default to `SOAP Client`)
  - create mode (`configured` or `scaffold-only`)
  - endpoint template/value
  - SOAP request XML
  - request mime type (default `text/xml`)
  - SOAPAction
  - HTTP authentication settings
  - HTTP headers such as `Accept` or custom service headers
  - expected response code strategy
  - notes
  - datasource binding when request values are supplied from datasource-backed variables
  - deterministic placement index in suite
  - copy target parent/name

### 4.0) Create Mode Selection (Required)
- Two create modes are supported:
  - `configured` (default): create a ready-to-run SOAP Client with endpoint + request XML on the validated HTTP branch.
  - `scaffold-only`: create an empty/minimal SOAP Client shell and defer operational fields.
- If the user does not explicitly request scaffold mode, proceed as `configured`.

### 4.1) Input Gate (Required)
- Never infer or silently apply endpoint, SOAPAction, request XML, headers, auth, or expected response values from prior examples/runs.
- "Provided input" may come from either:
  - direct user input, or
  - orchestration-provided context or active project record values that have been explicitly confirmed by the user for this branch.
- If create/update intent is underspecified, pause mutation and solicit missing values.
- In project-aware branches, consult active project facts/references/environment files for candidate endpoints, SOAPAction defaults, auth/header variables, and known request-value anchors before asking the user to restate them, but do not silently treat those candidates as approved.
- Do not switch to scaffold mode implicitly.

### 4.2) Conditional Required Matrix
- Create mode:
  - `configured` by default
  - `scaffold-only` only when user explicitly requests it
- Always required for create:
  - parent suite id
- Required for configured create on the validated HTTP branch:
  - endpoint template/value
  - SOAP request XML
- Optional for configured create:
  - SOAP Client name
  - SOAPAction
  - auth settings
  - non-owned HTTP headers
  - notes
  - expected response code strategy
- Required for negative/security-focused tests before final verification:
  - explicit expected response code strategy (`misc.validHttpResponseCodes`)

### 4.3) Missing-Field Solicitation Contract
When required fields are missing for the selected create mode, ask for all unresolved values in one prompt:
- Create mode (`configured` or `scaffold-only`; default `configured`):
- SOAP Client name (optional - I'll infer one if the operation name is clear):
- Endpoint URL/template:
- SOAP request XML:
- SOAPAction (optional):
- HTTP authentication (optional):
- Additional headers such as `Accept` (optional):
- Expected response codes (optional - I'll default to transport-default behavior unless you tell me otherwise):
Do not call create/update endpoints until required fields for the selected mode are provided/confirmed.

### 4.4) Scaffold-Only Create Mode
- Supported only when user explicitly requests blank/scaffold SOAP Client creation.
- In scaffold mode:
  - create with minimal shell payload (`parent.id` + optional `name`),
  - report configuration as incomplete immediately,
  - list remaining fields needed for operational readiness.
- Do not enter scaffold mode implicitly.

## 5) Preconditions
- API reachable and authenticated.
- Target suite exists.
- If parameterized endpoint/request values are used, referenced variables/data bank columns exist at runtime.
- If ordered placement is required, current child list can be read first.

## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile E for tool mutation steps (`POST`/`PUT`/`DELETE` and suite reorder),
   - use Skill 050 Profile D for execution/traffic-observation steps.
1. Inspect suite children and capture ordering baseline:
   - `GET /v6/suites/testSuites?id=<suite-id>` (read `relationships.childrenRel`)
2. Create SOAP Client:
   - enforce Input Gate and Solicitation Contract before create.
   - branch by create mode:
     - configured (default):
       - require all configured-mode fields per Section 4.2 before create.
     - scaffold-only (explicit user request only):
       - allow minimal shell creation with deferred request/transport fields.
   - `POST /v6/tools/soapClients`
   - scaffold create may use:
     - `parent.id`
     - optional `name`
   - configured HTTP create typically includes:
     - `parent.id`
     - `name`
     - `request`
     - `transport`
     - optional `misc`
2.1 Resolve SOAP Client identity before create (idempotent create rule):
   - derive deterministic target id as `<suite-id>/<soap-client-name>`,
   - if that id exists, update it in place,
   - create via `POST /v6/tools/soapClients` only when target SOAP Client is absent,
   - do not create another SOAP Client with the same intent/name in the same parent.
3. Configure request/transport/misc:
   - `PUT /v6/tools/soapClients?id=<tool-id>`
   - **PUT safety rule (required):** perform read-merge-write for SOAP Client updates.
     - first read `GET /v6/tools/soapClients?id=<tool-id>`
     - merge intended edits into current config
     - submit merged PUT preserving existing `request`, `transport`, and `misc` unless intentionally changed
     - avoid sparse/name-only PUT payloads for SOAP Clients
   - validated HTTP-focused payload shape:
     - `request.inputMode = formInput`
     - `request.literal.mimeType = text/xml`
     - `request.literal.type = text`
     - `request.literal.text = <SOAP envelope XML>`
     - `transport.type = http11`
     - `transport.http11.general.endpoint.endpointType = custom`
     - `transport.http11.general.endpoint.value = <endpoint template/value>`
     - `transport.http11.general.soapAction = <SOAPAction>`
     - `transport.http11.general.expectSynchronousResponse = true` by default
     - `transport.http11.general.connectionSettings = keepAlive` by default
     - `transport.http11.security.httpAuthentication` when auth is needed
     - `transport.http11.httpHeaders` for non-owned headers
     - `misc.notes`
     - `misc.timeout`
     - `misc.validHttpResponseCodes`
   - header ownership rule for client tools:
     - do **not** explicitly set `Content-Type` header for SOAP requests.
     - set `request.literal.mimeType` and allow SOAtest client tooling to emit the correct `Content-Type` automatically.
     - for broader client-header ownership policy and cross-client consistency, follow `docs/skills/cross-cutting/skill-032-client-header-ownership.md`.
   - WSDL-originated compatibility rule:
     - the API may preserve runnable behavior for WSDL-originated SOAP Clients without exposing first-class WSDL/XML Schema resource-mode fields in `GET /v6/tools/soapClients`.
     - when editing such tools, limit mutations to the exposed `request`, `transport`, and `misc` fields unless broader WSDL-tab support has been separately validated.
4. Read back and verify the tool:
   - `GET /v6/tools/soapClients?id=<tool-id>`
   - outside scaffold mode, verify request XML is still present
   - verify transport type and endpoint settings match intent
   - if the tool is WSDL-originated or copied from one, do not expect dedicated WSDL-tab state to appear in API readback; verify runnable endpoint/request behavior instead.
5. Optional copy flow (substep only):
   - for pure copy-only intent, prefer `docs/skills/structure/skill-070-copy-tool.md`
   - keep this branch only when copy is subordinate to broader SOAP Client lifecycle/configuration work
   - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, optional `to.name`
   - verify copied tool via `GET /v6/tools/soapClients?id=<new-id>` and parent readback
6. Optional delete flow (substep only):
   - for pure delete-only intent, prefer `docs/skills/structure/skill-072-delete-tool.md`
   - keep this branch only when delete is subordinate to broader SOAP Client lifecycle/configuration work
   - `DELETE /v6/tools?id=<tool-id>`
   - verify deletion via descendants/children readback
7. Reorder tool to required position (if needed):
   - build full ordered child list
   - `PUT /v6/suites/children?id=<suite-id>` with complete `children[].id` sequence
8. Execute focused validation against selected tests by following the Skill 012 execution-triad contract.
   - use a workspace-valid `general.config` (for example `soatest.user://Example Configuration`)
   - scope execution to the target `.tst` plus `soatestOptions.testNames`
9. Use the Skill 012 results/traffic readback steps to verify target statuses and, when needed, capture the specific SOAP Client Traffic Viewer evidence.
10. For negative/security-style tests, calibrate expected response-code behavior from runtime evidence before final verification:
   - inspect observed HTTP status codes from test-execution traffic
   - update `misc.validHttpResponseCodes` when intent requires non-default acceptance
   - note that HTTP `200` does not imply business success for SOAP payloads; interpret response body and chained XML-tool outcomes as well.

## 6.0) Authoring Rule (API-First)
- Follow global authoring + exemplar-efficiency policy in:
  - `docs/workflow/agent-workflow.md` (Decision Rule).
- Build SOAP Client payloads from user input + REST API schema + this card's validated field model.

## 6.0.1) Minimal Payload Shape Example (Disambiguation Only)
This example is shape-only and must not be copied with literal values.

Rules:
- Replace all `{{...}}` placeholders from user-provided values and confirmed runtime intent.
- Do not reuse sample ids/names/endpoints/request XML literals unless explicitly requested.
- For updates, use `PUT /v6/tools/soapClients?id=...` with GET -> mutate -> PUT read-merge-write per Skill 049.

Configured create shape:
```json
{
  "parent": {
    "id": "{{PARENT_SUITE_ID}}"
  },
  "name": "{{SOAP_CLIENT_NAME}}",
  "request": {
    "inputMode": "formInput",
    "literal": {
      "mimeType": "text/xml",
      "type": "text",
      "text": "{{SOAP_REQUEST_XML}}"
    }
  },
  "transport": {
    "type": "http11",
    "http11": {
      "general": {
        "soapAction": {
          "type": "fixed",
          "fixed": "{{SOAP_ACTION_OR_EMPTY}}"
        },
        "endpoint": {
          "endpointType": "custom",
          "value": {
            "type": "fixed",
            "fixed": "{{ENDPOINT_TEMPLATE_OR_LITERAL}}"
          }
        },
        "expectSynchronousResponse": true,
        "connectionSettings": "keepAlive"
      },
      "httpHeaders": {
        "type": "table",
        "httpHeadersTable": {
          "rows": []
        }
      }
    }
  },
  "misc": {
    "notes": "{{OPTIONAL_NOTES}}",
    "timeout": {
      "action": "failOnTimeout",
      "milliseconds": {
        "mode": "Default",
        "value": 30000
      }
    },
    "validHttpResponseCodes": {
      "type": "fixed",
      "fixed": "{{EXPECTED_CODES_OR_EMPTY}}"
    }
  }
}
```

Update shape (read-merge-write envelope):
```json
{
  "name": "{{SOAP_CLIENT_NAME}}",
  "dataSource": "{{OPTIONAL_DATASOURCE_NAME}}",
  "request": {
    "inputMode": "formInput",
    "literal": {
      "mimeType": "text/xml",
      "type": "text",
      "text": "{{SOAP_REQUEST_XML}}"
    }
  },
  "transport": {
    "type": "http11"
  },
  "misc": {
    "notes": "{{OPTIONAL_NOTES}}"
  }
}
```

## 7) Validation
- Expected HTTP status codes (API transport only):
  - create/update/read/reorder and execution/results calls typically return `200` on valid API requests.
- Expected response shape:
  - SOAP Client readback includes `request`, `transport`, and `misc`.
  - execution results include `summary` and `xmlReport` when requested.
- Post-condition checks:
  - tool appears under intended suite parent
  - endpoint/request XML settings match requested configuration
  - if create mode was scaffold-only, result is explicitly reported as incomplete with remaining fields listed
  - copied WSDL-originated SOAP Clients preserve visible request/transport/misc settings after copy
  - read-merge-write PUT of exposed fields preserves runnable behavior for copied WSDL-originated SOAP Clients in the validated HTTP branch
  - runtime interpretation includes chained XML-tool context so SOAP body/business failures are not reduced to HTTP status alone

## 8) Failure Modes
- `400` invalid payload shape or field incompatibility
- `404` invalid id or missing execution configuration during validation
- sparse PUT overwrite risk: partial SOAP Client PUT payload can clear required request/transport fields
- request-header drift risk: manually forcing `Content-Type` can conflict with SOAP Client managed wire-level headers
- WSDL visibility gap:
  - API readback does not currently expose first-class WSDL/XML Schema resource-mode/url fields even when a SOAP Client originated from WSDL-driven setup
- transport-surface gap:
  - the schema advertises a broader `soapTransportType` enum than the currently validated authorable payload branches
- misleading transport success:
  - HTTP `200` can still return a SOAP body indicating business failure or a chained XML-tool assertion failure
- output-mapping drift:
  - XML validation/data-exchange tools should chain to `Response SOAP Envelope`, not `Traffic Object`

## 9) Safety / Rollback
- Write skill (mutates `.tst` tool structure/configuration).
- Rollback options:
  - delete newly created/copied tools via `DELETE /v6/tools?id=<tool-id>`
  - restore prior suite order from saved `children` snapshot
  - reapply previous tool configuration via saved GET payload

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
- Related endpoints:
  - `POST/PUT/GET /v6/tools/soapClients`
  - `DELETE /v6/tools`
  - `POST /v6/tools/copy`
  - `PUT /v6/suites/children`
  - `POST /v6/testExecutions`
  - `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
  - `GET /v6/testExecutions/{id}/traffic?entityId=...`
- Validated output-parent evidence:
  - semantic response output: `<soap-client-id>/Response SOAP Envelope`
  - diagnostics output: `<soap-client-id>/Traffic Object`
- WSDL-originated compatibility note:
  - copy + read-merge-write PUT preserved runnable behavior for a WSDL-originated SOAP Client in the current workspace,
  - full API control over WSDL/XML Schema UI tabs remains unproven and is intentionally out of scope for this first-pass card.
- For pure rename/copy/delete/enable-disable prompts on an existing SOAP Client, prefer the centralized operation-centric owners; keep Skill 034 for broader SOAP Client lifecycle/configuration work.
