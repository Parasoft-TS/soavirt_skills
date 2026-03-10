# Skill 059: Existing Constrained REST Client Same-Operation Edit Workflow (API-First)

## 1) Skill Name
Safely edit existing service-definition-backed constrained REST Client request values while keeping the selected operation unchanged, using REST Client API updates for constrained JSON request payloads and YAML fallback for resource/config surfaces.

## 2) Objective
Provide a reliable same-operation edit path for existing constrained REST Clients. For constrained `Form JSON` request bodies, this card treats `GET /v6/tools/restClients` plus `PUT /v6/tools/restClients` as the authoritative way to write schema-valid JSON literal payloads and let the server normalize them into the persisted constrained `Form JSON` model. For same-operation resource/config surfaces such as base URL, schema URL, path parameters, and query parameters, this card keeps the existing YAML fallback path.

This card owns same-operation edits for existing constrained REST Clients only. It does not broaden into operation retargeting, cross-spec migration, or creating new constrained REST Clients.

## 3) Scope
- In scope:
  - existing REST Clients whose API readback indicates service-definition backing (for example `resource.type=swagger`) and whose persisted YAML indicates constrained mode (`resourceMode: 3`)
  - same-operation edits to existing constrained request values for:
    - path parameters
    - query parameters
    - JSON request body values for constrained `Form JSON` clients, using the selected operation's request schema to construct valid JSON literal text and the REST Client API to normalize the persisted constrained payload state
  - semantically equivalent literalization of:
    - `baseUrl.fixedValue`
    - `schemaURL.MessagingClient_SchemaLocation`
    - `serviceDescriptor.location`
    when the replacement still points to the same resolved base URL or same resolved service-definition location
  - REST Client `GET/PUT/GET` read-merge-write updates for constrained JSON request payloads
  - download/edit/upload/re-download/focused-execution verification for same-operation resource/config edits
- Out of scope:
  - creating new constrained REST Clients
  - changing the selected constrained operation
  - changing `resourceMethod.resourceId`
  - changing `resourceMethod.httpMethod`
  - structural YAML edits such as create/delete/move/reorder of tests, tools, parameters, or suites
  - switching payload modes away from constrained `Form JSON`
  - non-JSON constrained payload modes
  - OpenAPI v1-to-v2 migration orchestration and downstream chained-tool rewrites

## 3.1) Dependencies
- Required:
  - `docs/skills/platform/skill-002-shared-file-transfer.md`
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
  - `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`
- Additive:
  - `docs/skills/cross-cutting/skill-052-tst-configuration-analysis-dataflow-trace.md`

## 4) Inputs
- Required:
  - target `.tst` file id/path
  - target constrained REST Client id or unique test/tool name
  - intended same-operation request-value edits
- Optional:
  - local backup/output paths under `work/`
  - request to literalize environment-backed base URL or schema URL values without changing semantics
  - focused verification test name if the surrounding `.tst` contains multiple similarly named tests

## 5) Preconditions
- API reachable and authenticated.
- Target `.tst` and target REST Client are resolvable.
- API-first readback confirms the target is an existing constrained REST Client:
  - REST Client GET indicates service-definition backing (for example `resource.type=swagger`), and
  - downloaded YAML confirms constrained-mode structures for the selected tool block.
- Requested edits stay within the current selected operation and do not require structural mutation.
- For the constrained JSON body branch:
  - the selected operation's request schema/service definition is resolvable,
  - the intended request payload can be represented as valid JSON for that schema,
  - payload values follow the sourcing and approval ladder defined by `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`.

## 6) Procedure
0. Apply capability preflight before first write:
   - use Skill 050 Profile A/E to resolve the target file/tool and write branch,
   - use Skill 050 Profile C for download/upload verification,
   - use Skill 050 Profile D for focused verification execution.
1. Resolve the target tool through API reads first:
   - locate the `.tst` with `GET /v6/descendants/files`
   - locate the tool graph with `GET /v6/descendants/assets?id=<tst-id>`
   - read the current tool with `GET /v6/tools/restClients?id=<tool-id>`
2. Confirm that the edit belongs to this card:
   - existing constrained REST Client
   - same selected operation
   - request-value edits confined to the same-operation surfaces for this card
3. Classify the requested edit surface:
   - path-parameter edits -> `constrainedPath.pathParameters`
   - query-parameter edits -> keep all mirrored constrained-query fields synchronized:
     - `router.HTTPClient_Endpoint`
     - `urlParameters.properties`
     - `constrainedQuery.parameters`
   - constrained JSON body edits on constrained `Form JSON` clients -> derive the request payload from the selected operation's request schema, construct valid JSON literal text, and write it back through the REST Client API so the server normalizes the authoritative constrained payload state
   - semantically equivalent literalization -> only when replacing an environment-backed value with the same resolved literal value
4. For path/query/resource/config edits, download the `.tst` using Skill 002 and immediately keep the original downloaded file as rollback source.
5. For path/query/resource/config edits, locate the exact YAML block for the target REST Client.
  - anchor the block with stable identifiers from the current tool, including tool name plus operation-binding fields such as `resourceMethod` and `resourceMode`,
  - fail closed if the target block cannot be isolated unambiguously.
6. Apply the safe-edit envelope for YAML-based path/query/resource/config edits:
   - confine edits to one resolved target tool block at a time
   - use block-local replacement or parser-aware editing; do not use whole-file/global regex or broad find/replace as the default strategy
   - edit scalar values only within the existing target tool block
   - preserve indentation, ordering, `$type` declarations, and list lengths
   - do not add/remove sibling parameter nodes, schema nodes, or operation-binding nodes
   - do not modify:
     - `testID`
     - `resourceMethod.resourceId`
     - `resourceMethod.httpMethod`
     - `resourceMode`
     - output provider blocks
     - suite/test/tool structure
7. For constrained JSON payload edits:
   - read the current tool with `GET /v6/tools/restClients?id=<tool-id>`
   - keep the full GET payload as the mutation base; do not send sparse/name-only PUT payloads
   - derive the request payload shape from the selected operation's request schema/OpenAPI definition
   - source payload values under the Skill 058 ladder:
     - explicit user-provided values
     - values already confirmed earlier in the current session
     - contract hints such as examples, defaults, enums, types, requiredness, and field structure
     - values inferred from other tests or datasources in the same `.tst`
   - when the current remediation mode does not allow autonomous filling, treat generated/best-guess values as proposals and require the existing remediation approval gate before writing them
   - construct valid JSON literal text that conforms to the selected operation's request schema
   - update `payload.input.literal.text` in the live GET object
   - write the full updated tool back with `PUT /v6/tools/restClients?id=<tool-id>`
   - treat the server-side REST Client PUT path as the authoritative normalizer for converting valid schema-conformant JSON literal text into the persisted constrained `Form JSON` model
8. For YAML-based path/query/resource/config edits, write the edited file as UTF-8 without BOM.
9. For YAML-based path/query/resource/config edits, compare the target tool block locally before upload and confirm only the intended request-value surfaces changed.
10. For YAML-based path/query/resource/config edits, upload the edited file with `replace=true`.
11. Re-read the updated state and confirm that:
   - for API-based payload edits, REST Client GET readback reflects the intended payload JSON
   - for YAML-based path/query/resource/config edits, intended edits persisted
   - for constrained JSON payload edits, the re-downloaded `.tst` shows the server-normalized constrained payload state, including persisted `formJson` and `MessagingClient_LiteralMessage`
   - for YAML-based edits, all required mirrored surfaces changed together
   - no unrelated drift appeared in the edited block or neighboring tool structure.
12. For constrained payload edits, verify the actual request payload sent on the wire rather than relying only on UI-facing literal text.
13. Run focused verification execution for the selected test and confirm the runtime effect matches the intended same-operation edit.
14. If verification fails:
   - stop
   - restore the original `.tst` or original tool JSON readback, depending on which path was used
   - report the failure so the next step can focus on payload construction or runtime behavior rather than speculative YAML structure edits

## 6.1) Safe YAML Edit Examples
### A) Path parameter edit inside the same operation
Update both the default and active value in the same existing constrained path parameter node:

```yaml
constrainedPath:
  pathParameters:
  - $type: ElementValue
    type:
      $type: ElementType
      defaultValue: 12212
      localName: customerId
      bodyType:
        $type: IntegerType
    values:
    - $type: IntegerValue
      value: 12212
```

Do not change the surrounding operation-selection fields for this Level 1 path.

### B) Query parameter edit inside the same operation
When a constrained request uses query parameters, keep all mirrored fields synchronized:

```yaml
router:
  fixedValue:
    HTTPClient_Endpoint: "${BASEURL}/deposit?accountId=12345&amount=1.0"
urlParameters:
  properties:
  - name: accountId
    value:
      fixedValue:
        value: 12345
  - name: amount
    value:
      fixedValue:
        value: 1.0
constrainedQuery:
  parameters:
  - $type: ElementValue
    qnameAsString: :accountId
    values:
    - $type: IntegerValue
      value: 12345
  - $type: ElementValue
    qnameAsString: :amount
    values:
    - $type: DecimalValue
      value: 1.0
```

Editing only one of those surfaces is unsafe/incomplete for this card.

### C) Safe literalization example
If an environment-backed value already resolves to a known literal and the edit is explicitly intended to keep semantics unchanged, this card may replace:
- `baseUrl.fixedValue.value: "${BASEURL}"` -> `baseUrl.fixedValue.value: "http://example/api"`
- `schemaURL.MessagingClient_SchemaLocation: "${OPENAPI}"` -> the same resolved OpenAPI URL
- `serviceDescriptor.location: <resolved openapi url>` -> `${OPENAPI}`

Only do this when the resolved location stays the same.

### D) Constrained JSON body edit through the REST Client API
For constrained `Form JSON` clients, construct valid JSON for the selected operation's request schema, then update the REST Client through read-merge-write:

```json
{
  "payload": {
    "input": {
      "literal": {
        "text": "{\"name\":\"Oz Payee\",\"address\":{\"street\":\"1 Main St\"},\"phoneNumber\":\"555-0100\",\"accountNumber\":12345}"
      }
    },
    "inputMode": "formJSON"
  }
}
```

After `PUT /v6/tools/restClients`, treat the server as authoritative for expanding and normalizing the persisted constrained `formJson` state, then prove the change through API readback, `.tst` re-download, and focused runtime traffic.

## 7) Validation
- Required checks:
  - target tool is still present under the same parent after update
  - API readback shows the intended same-operation request values
  - re-downloaded YAML shows the intended request-value changes and no unintended structural changes
  - focused verification execution proves the target test still runs and the intended request-value change takes effect
- Level 1 safety rules:
  - treat `resourceMethod.resourceId` as high-risk and non-editable for this card
  - do not claim support for operation retargeting from evidence gathered only from same-operation value edits
  - for constrained `Form JSON` payload changes, use REST Client API `GET/PUT/GET` rather than hand-building persisted constrained payload YAML
  - construct JSON that is valid and schema-conformant for the selected operation's request payload
  - trust the server-side REST Client PUT path to normalize valid schema-conformant JSON literal text into the authoritative persisted constrained payload model
  - treat the request schema and the approved Skill 058 sourcing ladder as the authoritative inputs for payload construction
  - verify payload changes through API readback, `.tst` readback, and runtime traffic
  - before upload, compare the local target block and fail closed if the edit touched unrelated tools or required broad regex drift

## 8) Failure Modes
- Upload saved with BOM (`EF BB BF`) corrupts the `.tst`.
- Editing only one mirrored query field leads to inconsistent persisted state.
- Sparse/name-only REST Client PUT drops required existing configuration.
- Constructed payload JSON is invalid or does not conform to the selected operation's request schema, so the server rejects or misinterprets it.
- Editing `resourceMethod.resourceId` or related operation-binding fields breaks execution even when request values are otherwise valid.
- Structural list edits or `$type` edits make the `.tst` unreadable in the UI.
- Whole-file or broad regex replacement causes unrelated tool drift that is hard to verify safely.
- Focused verification fails because the request payload values are wrong for the target scenario even though the constrained payload model normalized successfully.

## 9) Safety / Rollback
- Always keep the original downloaded `.tst` as the rollback source for YAML-based edits.
- Always keep the original full REST Client GET payload as the rollback source for API-based payload edits.
- Keep a local before/after snapshot of the target tool block so non-target drift can be detected before upload.
- If update verification or focused execution fails, restore the original file or tool payload immediately and re-read to confirm restoration.
- Keep all playground or experiment files under `work/`.

## 10) Reuse Notes
- Use this card only for existing constrained REST Clients whose selected operation remains unchanged.
- Route unconstrained REST Client lifecycle work to `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.
- Route broader request-readiness orchestration to `docs/skills/composite-orchestration/skill-058-request-readiness-remediation-orchestration.md`.
- Treat operation retargeting, cross-spec migration, and new constrained-client creation as future higher-scope work, not as implied extensions of this card.
- For constrained JSON request payloads, this card is the preferred path because the server-side REST Client API normalizes valid schema-conformant JSON into the persisted constrained `Form JSON` model.

## 11) Reliability Boundary
- This card treats same-operation constrained JSON request-body editing through the REST Client API as a reliable path for existing constrained `Form JSON` REST Clients.
- The authoritative rule for payload construction is:
  - derive the request payload shape from the selected operation's request schema/OpenAPI definition
  - construct valid JSON that conforms to that schema
  - write it through REST Client `GET/PUT/GET`
  - trust the server to normalize it into the persisted constrained `Form JSON` model
- This card also supports same-operation constrained path/query/resource/config edits through the YAML path documented above.
- This card does not authorize constrained operation retargeting, new constrained-client creation, non-JSON constrained payload modes, or end-to-end OpenAPI migration work.
