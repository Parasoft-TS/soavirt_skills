# Skill 056: Single-Client Authoring Intent Orchestration (Composite)

## 1) Skill Name
Clarify and execute single REST/SOAP client authoring intent across endpoint-only and contract-informed branches.

## 2) Objective
Convert requests whose normalized intent is "create one client/test" into a confirmed execution plan that uses service definitions as planning input when helpful, but routes final authoring to the smallest matching client-lifecycle skill.

## 3) Scope
- In scope:
  - normalize single-client intent versus broad generation intent
  - classify REST vs SOAP branch
  - support endpoint-only and contract-informed planning branches
  - use OpenAPI/WSDL as planning input for one operation/client when available
  - resolve whether to place the client into an existing asset or create a minimal new host asset first
  - route REST execution to Skill 020
  - route SOAP execution to Skill 034
  - optionally offer or hand off follow-on validation enrichment after successful one-client authoring/execution
- Out of scope:
  - broad multi-operation generation from service definitions
  - claiming durable OpenAPI/WSDL-bound client constraints after authoring
  - replacing the endpoint-accurate behavior owned by Skills 020 and 034

## 3.1) Dependencies
- Required:
  - `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`
- Additive:
  - `docs/skills/platform/skill-021-tst-create-empty.md`
  - `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md`
  - `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`
  - `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`

## 4) Inputs
- Required:
  - user objective to create exactly one REST/SOAP client or one operation-focused test
  - at least one initial anchor:
    - service endpoint,
    - service definition/schema location,
    - explicit operation name/selection
- Conditionally required before first write:
  - protocol branch (`REST` or `SOAP`) when it cannot be inferred confidently
  - target hosting decision:
    - existing `.tst` / suite, or
    - create a minimal new host asset
  - operation selection when the source describes multiple operations
  - client-specific required fields owned by the downstream leaf skill
- Optional:
  - preferred service-definition format (`openapi`, `wsdl`)
  - payload/value proposal mode
  - validation-tooling preferences
  - follow-on validation-enrichment preference when already resolved upstream

## 5) Preconditions
- API reachable and authenticated for all downstream skills.
- User intent is normalized to one-client authoring rather than broad contract generation.
- Target host asset can be resolved or safely created before client creation.

## 6) Procedure
### 6.1) Intent Normalization
1. Confirm that the user wants one client/test rather than broad generated coverage.
2. If the user actually wants multi-operation generation, reroute:
   - underspecified broad authoring -> Skill 033
   - direct file-level generation -> Skills 022-025
   - direct existing-asset WSDL suite generation -> Skill 055
3. Classify protocol branch:
   - REST
   - SOAP
4. Classify source branch:
   - endpoint-only
   - contract-informed planning

### 6.2) Contract-Informed Planning Branch
5. If a service definition is provided, use it as planning input rather than as a promise of durable tool binding:
   - summarize available operations,
   - let the user select one operation when multiple are present,
   - extract candidate method/path/request-body needs for REST,
   - extract candidate operation/request-envelope/endpoint hints for SOAP.
6. For REST:
   - treat OpenAPI as planning input for one client, then route final authoring to Skill 020.
7. For SOAP:
   - treat WSDL as planning input for one client, then route final authoring to Skill 034.
   - explicitly preserve Skill 034's boundary: this is contract-informed SOAP Client authoring, not full WSDL-bound SOAP Client parity.

### 6.3) Host Asset Resolution
8. Resolve where the client should live:
   - existing `.tst` + target suite, or
   - create a minimal new `.tst` host via Skill 021, then use the root `Test Suite` or create/select a non-root hosting suite via Skill 009 when needed.
9. Resolve any missing target ids with API-first discovery before client creation.

### 6.4) Downstream Handoff
10. For REST single-client execution:
   - route to `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.
11. For SOAP single-client execution:
   - route to `docs/skills/client-tools/skill-034-soap-client-http-lifecycle.md`.
12. Carry forward only explicitly confirmed values from intake/contract planning into the downstream leaf skill.

### 6.4.1) Optional Post-Execution Validation-Enrichment Handoff
13. After the downstream client skill has created and executed the selected one-client test successfully enough to produce runtime evidence:
  - if follow-on validation-enrichment preference was already resolved upstream (for example by Skill 033), honor that resolved value and do not ask again.
  - if no follow-on validation was approved upstream, finish the one-client branch cleanly without reopening broader phase-sequencing questions.
14. If follow-on validation-enrichment was already approved upstream:
  - hand off directly to `docs/skills/composite-orchestration/skill-057-validation-enrichment-intent-orchestration.md`.
15. If this card was reached directly from `docs/skills/skill-index.md` and follow-on validation-enrichment preference is still unresolved:
  - offer one optional follow-up question after reporting successful one-client authoring/execution:
    - "Do you want to add validation to this test now?"
16. If the user accepts that follow-on step:
  - hand off to Skill 057 using the just-created test and its runtime evidence as the starting context.
17. If the user declines:
  - stop cleanly without reopening broader intake.

### 6.5) Confirmation Rule
18. Before first write, summarize:
   - selected protocol branch,
   - endpoint-only vs contract-informed branch,
   - chosen operation,
   - host asset/parent target,
   - payload/value proposal mode,
   - downstream execution skill,
   - follow-on validation-enrichment preference when already known.
19. Require explicit user confirmation before writes.

## 7) Validation
- One-client intent is normalized before execution.
- Multi-operation generation requests are rerouted rather than forced through this card.
- REST leaf execution always ends in Skill 020.
- SOAP leaf execution always ends in Skill 034.
- Contract-informed SOAP wording does not overclaim full WSDL-bound authoring parity.
- Optional follow-on validation-enrichment handoff does not duplicate a preference already resolved upstream or reopen broader sequencing owned by Skill 033.

## 8) Failure Modes
- Broad generation intent is misclassified as one-client intent.
- Contract-informed planning is mistaken for durable contract binding.
- Target hosting asset remains unresolved before client creation.
- Operation-selection ambiguity is skipped when the source describes multiple operations.
- SOAP branch overclaims WSDL-constrained behavior that Skill 034 does not actually validate.
- Direct one-client routing misses the post-execution opportunity to offer validation enrichment.
- Skill 056 re-asks for follow-on validation when Skill 033 already resolved that preference upstream.

## 9) Safety / Rollback
- Orchestration-first policy: avoid speculative writes before confirmation.
- Rollback is delegated to downstream asset-creation and client-lifecycle skills used in the selected branch.

## 10) Reuse Notes
- Use this card for direct single-client requests and for contract-informed one-operation requests.
- Use Skill 033 instead when service-test authoring intent is still materially underspecified.
- Use generation cards (`Skills 022-025`, `Skill 055`) when the user wants broad generated coverage rather than one client.
- This card may optionally hand off to Skill 057 after successful one-client execution when the user wants immediate validation enrichment.
- Generalized readiness remediation and broader negative/validation phase sequencing remain owned by Skills 033 and 058 rather than by this card.
