# Skill 059: Single Constrained REST Client Lifecycle (YAML Promotion + API Body Normalization)

## 1) Skill Name
Create/read/update/copy/delete one service-definition-backed constrained REST Client, using shell-first YAML promotion for constrained binding and REST Client API `GET/PUT/GET` normalization for constrained JSON request bodies.

## 2) Objective
Provide one reliable v1 owner for single constrained REST Client lifecycle work. This card supports:
- creating a fresh constrained REST Client from scratch
- reading, copying, deleting, and same-client updating one existing constrained REST Client
- same-operation request-value edits on an existing constrained client
- JSON body-bearing create/update flows only through the validated two-phase path:
  - minimal YAML promotion into constrained mode
  - REST Client API body normalization through read-merge-write `PUT`

Operation identity is defined by OpenAPI path plus HTTP method, not by `operationId`. This card fail-closes on broader multi-client orchestration, speculative environment-variable invention, non-JSON constrained body modes, and unvalidated operation-retargeting flows outside the shell/promotion workflow documented here.

## 3) Scope
- In scope:
  - one REST Client whose API readback indicates service-definition backing (for example `resource.type=swagger`) and whose persisted YAML indicates constrained mode (`resourceMode: 3`)
  - read via `GET /v6/tools/restClients?id=...`
  - create a fresh constrained REST Client by:
    - creating or reusing a plain REST Client shell
    - promoting it through isolated YAML edits inside one target tool block
    - optionally normalizing a JSON request body through REST Client `GET/PUT/GET`
  - update of one existing constrained REST Client when the work stays within the same selected operation
  - copy via `POST /v6/tools/copy`
  - delete via `DELETE /v6/tools?id=...`
  - same-operation edits to constrained request/config surfaces for one target tool:
    - path parameters
    - query parameters
    - semantically equivalent base URL / schema URL / service-definition literalization
    - constrained JSON request body values for constrained `Form JSON` clients
  - deterministic same-spec style reuse:
    - reuse existing schema/base-url variable-token style from same-spec constrained peers when that style is internally consistent
    - preserve hardcoded literal style when same-spec peers already use literals
    - allow no-peer fresh creation from explicit literals or caller-supplied variable tokens
- Out of scope:
  - broader multi-client refactor or migration orchestration
  - operation retargeting outside the validated shell/promotion workflow in this card
  - structural YAML edits outside one isolated target tool block
  - autonomous invention of new environment-variable names
  - switching constrained payload modes away from JSON `Form JSON`
  - non-JSON constrained payload modes

## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
  - `docs/skills/platform/skill-003-server-copy.md`
  - `docs/skills/platform/skill-006-safe-local-yaml-edit-composite.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` when the task or owning workflow explicitly includes runtime execution/traffic verification
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`
  - `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`

## 4) Inputs
- Required:
  - target `.tst` file id/path
  - target parent suite id when creating or copying
  - target constrained REST Client id or unique test/tool name for read/update/delete
  - OpenAPI/schema location and base URL source for fresh constrained creation when they cannot be reused from a same-spec peer
  - selected OpenAPI path plus HTTP method
- Optional:
  - REST Client/test name
  - caller-supplied variable tokens for base URL or schema URL
  - intended path/query/body request values
  - local backup/output paths under `work/`
  - focused verification test name if the surrounding `.tst` contains similarly named tests

## 5) Preconditions
- API reachable and authenticated.
- Target `.tst` and target suite/tool are resolvable.
- For update branches, API readback plus downloaded YAML confirm the target is already a constrained REST Client.
- For create branches, the intended OpenAPI path plus HTTP method are known before mutation.
- For JSON body-bearing branches:
  - the selected operation's request schema/service definition is resolvable
  - the intended payload can be represented as valid JSON for that schema
  - payload values follow the sourcing and approval ladder in `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`

## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile A/E for file/tool resolution and mutation steps
   - use Skill 050 Profile C for download/upload verification
   - use Skill 050 Profile D only when the task or owning workflow explicitly includes focused execution and traffic observation
1. Resolve the branch before mutation:
   - read existing constrained client
   - create a fresh constrained client
   - update one existing constrained client without changing the selected operation
   - copy one constrained client
   - delete one constrained client
2. Resolve the file, suite, tool, and operation context through API reads first:
   - locate the `.tst` with `GET /v6/descendants/files`
   - inspect parent/tool graph with `GET /v6/descendants/assets?id=<tst-id>`
   - for existing tools, read the current tool with `GET /v6/tools/restClients?id=<tool-id>`
   - identify the target constrained operation from the OpenAPI path plus HTTP method; do not treat `operationId` as the authoritative binding key
3. Determine the schema/base-url style before editing:
   - if same-spec constrained peers exist and consistently use `${TOKEN}` style for schema/base URL, reuse that exact style
   - if same-spec constrained peers consistently use hardcoded literals, preserve that literal style
   - if no same-spec constrained peer exists, proceed with explicit literal values or caller-supplied variable tokens
   - never invent a new environment-variable name autonomously
4. Read branch:
   - use `GET /v6/tools/restClients?id=<tool-id>`
   - confirm constrained/service-definition-backed state through API plus YAML evidence
5. Copy branch:
   - `POST /v6/tools/copy` with `from.id`, `to.parent.id`, and optional `to.name`
   - verify the copied tool through `GET /v6/tools/restClients?id=<new-id>` plus parent readback
6. Delete branch:
   - `DELETE /v6/tools?id=<tool-id>`
   - verify deletion through descendants/children readback
7. For create or YAML-surface update branches, invoke Skill 006 to establish rollback-preserving local YAML edit handling:
   - create a durable server-side fallback copy when needed
   - download the `.tst`
   - preserve the original local rollback source
   - isolate one target tool block before editing
8. Create/update the constrained YAML block with a shell-first envelope:
   - for fresh creation:
     - create or reuse a plain REST Client shell via `POST /v6/tools/restClients`
     - use the shell as the target for constrained promotion; do not require copying an existing constrained client
   - for existing-client same-operation YAML edits:
     - keep the current selected operation unchanged
   - in the isolated target tool block, set or preserve the minimum proven constrained structures:
     - aligned tool/test name
     - `serviceInfo.serviceDescriptor.location`
     - `schemaURL.MessagingClient_SchemaLocation`
     - `baseUrl.fixedValue.value`
     - `router.HTTPClient_Endpoint`
     - `literalPath`
     - `resourceMethod.resourceId = <OpenAPI path>`
     - `resourceMethod.httpMethod = <HTTP method>`
     - `resourceMode: 3`
   - path-plus-method is the authoritative operation binding rule for this card
   - keep edits block-local:
     - do not use whole-file/global regex as the default strategy
     - preserve indentation, ordering, `$type` declarations, and list lengths
     - do not modify neighboring tools or suite structure
9. For path/query/resource/config surfaces in the YAML branch:
   - path parameter edits stay within the existing `constrainedPath.pathParameters` nodes
   - query parameter edits must keep all mirrored surfaces synchronized:
     - `router.HTTPClient_Endpoint`
     - `urlParameters.properties`
     - `constrainedQuery.parameters`
   - semantically equivalent literalization is allowed only when the resolved value stays the same
   - optional query parameters may remain structurally present but disabled/unset by default
10. For JSON body-bearing create/update branches, use the required second phase after YAML promotion:
   - read the live tool with `GET /v6/tools/restClients?id=<tool-id>`
   - keep the full GET payload as the mutation base; do not send sparse/name-only PUT payloads
   - derive the request payload shape from the selected operation's request schema/OpenAPI definition
   - source payload values under the Skill 058 ladder:
     - explicit user-provided values
     - values already confirmed earlier in the current session
     - contract hints such as examples, defaults, enums, requiredness, and field structure
     - values inferred from other tests or datasources in the same `.tst`
   - when autonomous filling is not authorized, treat generated values as proposals and obtain the existing approval gate before writing them
   - set:
     - `payload.inputMode = formJSON`
     - `payload.input.literal.text = <valid schema-conformant JSON>`
   - write the full object back with `PUT /v6/tools/restClients?id=<tool-id>`
   - treat the server-side REST Client PUT path as the authoritative normalizer for persisted constrained `formJson`
11. Write the YAML-edited file as UTF-8 without BOM and confirm locally that only the intended target block changed before upload.
12. Re-read the updated state and verify all required persisted-state evidence:
   - API readback still resolves the tool under the intended parent
   - YAML readback shows the intended constrained operation binding and request-value state
   - for JSON body-bearing branches, API readback shows `payload.inputMode = formJSON`
   - for JSON body-bearing branches, re-downloaded YAML shows persisted `formJson` plus `MessagingClient_LiteralMessage`
   - enclosing suite and tool nodes remain resolvable after create/update
12.1. Optional runtime verification branch:
   - run this branch only when the user explicitly asks for runtime verification or the owning workflow explicitly includes it
   - focused execution proves the test still runs
   - runtime traffic proves the intended request path/query/body is sent on the wire
13. If required persisted-state verification fails:
   - stop
   - restore the original `.tst` through Skill 006 for YAML-based failure
   - restore the original full REST Client GET payload for API-body failure
   - report the exact failing verification gate rather than speculating about unsupported structure edits

## 6.1) Proven Branch Examples
### A) Fresh query-only constrained create with no same-spec peer
Use explicit literals or caller-supplied tokens and bind the operation by path plus method:

```yaml
baseUrl:
  fixedValue:
    value: "${BASEURL}"
schemaURL:
  MessagingClient_SchemaLocation: "${OPENAPI}"
serviceInfo:
  serviceDescriptor:
    location: "${OPENAPI}"
router:
  fixedValue:
    HTTPClient_Endpoint: "${BASEURL}/deposit?accountId=12345&amount=1.0"
literalPath: /deposit
resourceMethod:
  resourceId: /deposit
  httpMethod: GET
resourceMode: 3
```

If no same-spec peer exists, this branch remains valid; do not fail solely because there is nothing to compare against.

### B) Fresh path-plus-query constrained create reusing same-spec style
When same-spec peers consistently reuse the same tokens, preserve that pattern while keeping mirrored query surfaces synchronized:

```yaml
router:
  fixedValue:
    HTTPClient_Endpoint: "${BASEURL}/customers/12212?verbose=true"
constrainedPath:
  pathParameters:
  - $type: ElementValue
    type:
      $type: ElementType
      defaultValue: 12212
      localName: customerId
    values:
    - $type: IntegerValue
      value: 12212
urlParameters:
  properties:
  - name: verbose
    value:
      fixedValue:
        value: true
constrainedQuery:
  parameters:
  - $type: ElementValue
    qnameAsString: :verbose
    values:
    - $type: BooleanValue
      value: true
```

Editing only one query mirror is unsafe for this card.

### C) Fresh JSON body-bearing constrained create
Phase 1: minimally promote the shell in YAML:

```yaml
resourceMethod:
  resourceId: /v1/demoAdmin/preferences
  httpMethod: PUT
resourceMode: 3
router:
  fixedValue:
    HTTPClient_Endpoint: "${BASEURL}/v1/demoAdmin/preferences"
```

Phase 2: normalize the constrained JSON body through REST Client API read-merge-write:

```json
{
  "payload": {
    "inputMode": "formJSON",
    "input": {
      "literal": {
        "text": "{\"industryType\":\"DEFENSE\",\"webServiceMode\":\"REST_API\",\"advertisingEnabled\":false}"
      }
    }
  }
}
```

After `PUT /v6/tools/restClients`, prove the persisted result through API readback and `.tst` readback. Focused execution and runtime traffic are optional follow-up verification only when explicitly in scope. Do not try to hand-synthesize the persisted constrained `formJson` tree from scratch.

## 7) Validation
- Required checks:
  - target tool is present under the intended parent after create/update/copy
  - API readback shows service-definition backing and the intended operation binding
  - YAML readback shows the intended constrained structures without unrelated drift
  - for constrained `Form JSON` bodies, API/YAML readback confirms the intended persisted request payload shape
- Optional follow-up checks when runtime verification is explicitly in scope:
  - focused verification execution runs the target test
  - runtime traffic confirms the intended path/query/body on the wire
- Level 1 safety rules:
  - treat path plus method as the authoritative operation binding rule
  - do not claim support for arbitrary operation retargeting from evidence gathered only from the validated shell/promotion workflow
  - for constrained `Form JSON` bodies, use REST Client API `GET/PUT/GET` rather than hand-building persisted constrained payload YAML
  - before upload, compare the local target block and fail closed if the edit touched unrelated tools or required broad regex drift
  - fresh constrained creation must still succeed when no same-spec peer exists, provided the user or caller supplied the needed literal values or tokens
  - this card does not autonomously authorize runtime execution when the owning workflow stops at persisted-structure verification

## 8) Failure Modes
- Upload saved with BOM (`EF BB BF`) corrupts the `.tst`.
- YAML promotion binds the wrong operation because the selected OpenAPI path/method were not confirmed first.
- Query edits drift because only one mirrored field was updated.
- Sparse/name-only REST Client PUT drops required configuration.
- Constructed payload JSON is invalid or does not conform to the selected operation's request schema.
- Hand-built constrained payload trees or copied wizard blocks destabilize persisted structure.
- Broad regex or whole-file replacement causes unrelated tool drift.
- Focused execution or traffic verification is treated as mandatory even though the task or owning workflow only required persisted-structure verification.

## 9) Safety / Rollback
- Always route YAML-based rollback-preserving file-edit handling through Skill 006.
- Always keep the original full REST Client GET payload as the rollback source for API-body edits.
- Keep local before/after snapshots of the target tool block so non-target drift can be detected before upload.
- If verification fails, restore immediately and re-read to confirm restoration.
- Keep research artifacts and disposable experiments under `work/`.

## 10) Reuse Notes
- Use this card when the task is clearly about one constrained REST Client and the validated shell/promotion boundary is sufficient.
- Route unconstrained REST Client lifecycle work to `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.
- Route broader request-readiness orchestration to `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`.
- Route broader multi-client planning and ambiguity resolution to `docs/skills/composite-orchestration/skill-056-single-client-authoring-intent-orchestration.md` or `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md` when the user has not yet narrowed the task to one constrained client.
- Non-JSON constrained body modes remain future work.

## 11) Reliability Boundary
- This card treats single constrained REST Client lifecycle work as validated for:
  - read/copy/delete of one constrained client
  - same-operation edits on one existing constrained client
  - fresh constrained creation for non-body operations through shell-first YAML promotion
  - fresh constrained creation for JSON body-bearing operations only through minimal YAML promotion plus REST Client API body normalization
- The authoritative construction rule for constrained JSON bodies is:
  - derive payload shape from the selected operation's request schema/OpenAPI definition
  - construct valid schema-conformant JSON
  - write it through REST Client `GET/PUT/GET`
  - trust the server to normalize the persisted constrained `Form JSON` model
- The default completion gate for this card is persisted-state verification through API/YAML/file readback; execution and traffic observation are optional follow-up verification only when explicitly in scope.
- This card does not authorize broader multi-client migration/refactor orchestration, speculative new env-var naming, or non-JSON constrained payload authoring.
