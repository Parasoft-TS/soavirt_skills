# Skill Backlog

## Foundation (Shared REST Lifecycle)

1. List workspace roots and immediate children
- Endpoint: `GET /v6/children`
- Status: Defined in `docs/skills/platform/skill-001-shared-introspection.md`

2. Enumerate descendants by folder and type
- Endpoint: `GET /v6/descendants/files`
- Status: Defined in `docs/skills/platform/skill-001-shared-introspection.md`

3. Download file content by id
- Endpoint: `GET /v6/files/download`
- Status: Validated (no-op round-trip) in `docs/skills/platform/skill-002-shared-file-transfer.md`

4. Upload file content by id
- Endpoint: `POST /v6/files/upload`
- Status: Validated (no-op round-trip) in `docs/skills/platform/skill-002-shared-file-transfer.md`

5. Copy and rename file on server
- Endpoint: `POST /v6/files/copy`
- Status: Validated in `docs/skills/platform/skill-003-server-copy-rename.md`

6. Rename file in place on server
- Endpoint: `PUT /v6/files`
- Status: Validated in `docs/skills/platform/skill-004-server-rename.md`

7. Delete file on server
- Endpoint: `DELETE /v6/files`
- Status: Validated in `docs/skills/platform/skill-005-server-delete.md`

8. Safe file refactor composite workflow
- Endpoint: Composite of `POST /v6/files/copy` + `PUT /v6/files` + `DELETE /v6/files`
- Status: Defined in `docs/skills/platform/skill-006-safe-file-refactor-composite.md`

## Active Skill Card

- `docs/skills/cross-cutting/skill-007-tst-content-summarization.md`

## SOAtest-Specific (After foundation)

9. Summarize `.tst` contents with REST + YAML correlation
- Endpoint: `GET /v6/descendants/assets` + `POST /v6/assets/data` + `GET /v6/files/download`
- Status: In progress via `docs/skills/cross-cutting/skill-007-tst-content-summarization.md`

10. Identify and classify `.tst` assets under `/TestAssets`
11. Extract key structure from downloaded `.tst` YAML
12. Perform safe round-trip update on sandbox `.tst`

## L2 Skill Family: TST Object Manipulation

Family map:
- `docs/skills/structure/skill-family-tst-object-manipulation.md`

13. Atomic card: Reorder children in suite
- Endpoint: `PUT /v6/suites/children`
- Status: Endpoint behavior validated; card authoring pending

14. Atomic card: Move object to new parent
- Endpoint: `PUT /v6/suites/move` + related move endpoints by object class
- Status: Planned

15. Atomic card: Copy object by class
- Endpoint: class-specific copy endpoints where available
- Status: Planned

16. Atomic card: Rename object by class
- Endpoint: class-specific rename endpoints where available
- Status: Planned

17. Atomic card: Delete object by class
- Endpoint: class-specific delete endpoints where available
- Status: Planned

18. Overlay set: object-class constraints
- Classes: suites, tools, data sources, environments, global properties
- Status: Planned

19. Composite card: copy + move + reorder (paste at target position)
- Endpoint: Composite orchestration
- Status: Planned

20. Atomic card: Data source type-targeted move (generalized)
- Endpoints: `POST /v6/datasources/move` + YAML fallback via `GET /v6/files/download` / `POST /v6/files/upload`
- Status: Defined in `docs/skills/structure/skill-008-datasource-type-targeted-move.md`

21. Atomic card: Test suite creation and configuration (staged)
- Endpoints: `POST /v6/suites/testSuites` + `PUT /v6/suites/testSuites?id=...` + `GET /v6/suites/testSuites?id=...`
- Status: Defined in `docs/skills/structure/skill-009-testsuite-creation-and-configuration.md` (option matrix deferred for later validation)

22. Atomic card: JSON Assertor CRUD + copy
- Endpoints: `POST/PUT/GET /v6/tools/jsonAssertors` + `DELETE /v6/tools` + `POST /v6/tools/copy`
- Status: Defined in `docs/skills/validation/skill-010-json-assertor-workflow.md` (create validated; modify/delete/copy pending)

23. Cross-cutting card: XPath-over-JSON query semantics
- Scope: selector-expression fields on JSON tools (JSON Assertor, JSON Data Bank, and similar)
- Status: Defined in `docs/skills/cross-cutting/skill-011-xpath-over-json-query-semantics.md`

24. Atomic card: Server test execution + XML runtime report retrieval
- Endpoints: `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined and validated in `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md`

25. Cross-cutting card: deterministic test naming policy
- Scope: unique/stable naming for test and tool nodes to avoid `testNames` collisions and default labels (for example `REST Client`)
- Status: Defined in `docs/skills/cross-cutting/skill-013-test-naming-policy.md`

26. Atomic extension: test-execution traffic diagnostics retrieval
- Endpoints: `GET /v6/testExecutions/{id}/traffic?entityId=<test-suite-id|traffic-viewer-id>`
- Status: Defined and validated in `docs/skills/execution-diagnostics/skill-012-test-execution-xml-report.md` (section 8.2)

27. Atomic card: focused test-failure diagnosis using runtime traffic
- Endpoints: `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true` + `GET /v6/testExecutions/{id}/traffic`
- Status: Defined and validated in `docs/skills/execution-diagnostics/skill-014-test-failure-diagnostics-from-traffic.md`

28. Atomic card: DB Tool lifecycle (create/update/copy/move/delete)
- Endpoints: `POST/PUT/GET /v6/tools/dbTools` + `POST /v6/tools/copy` + `POST /v6/tools/move` + `DELETE /v6/tools` + `PUT /v6/suites/children`
- Status: Defined and validated in `docs/skills/client-tools/skill-015-db-tool-lifecycle.md`

29. Atomic card: XML-output Assertor workflow (cross-tool)
- Endpoints: `POST/PUT/GET /v6/tools/xmlAssertors` + `POST /v6/tools/move` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined and validated in `docs/skills/validation/skill-016-xml-assertor-workflow.md`

30. Cross-cutting card: output-provider chaining model
- Scope: tool chaining semantics across standalone tools and output channels; parent-anchor selection defaults.
- Status: Defined and validated in `docs/skills/cross-cutting/skill-017-output-chaining-model.md`

31. Cross-cutting card: tool output map cheat-sheet (living)
- Scope: compact, continuously updated map of tool outputs, semantic purpose, and default chaining anchors.
- Status: Defined and validated in `docs/skills/cross-cutting/skill-018-tool-output-map-cheat-sheet.md`

32. Atomic card: XML Data Bank extraction workflow (cross-tool)
- Endpoints: `POST/PUT/GET /v6/tools/xmlDataBanks` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined and validated in `docs/skills/data-exchange/skill-019-xml-databank-extraction-workflow.md`

33. Atomic card: REST Client workflow (Service Definition = None)
- Endpoints: `POST/PUT/GET /v6/tools/restClients` + `DELETE /v6/tools` + `PUT /v6/suites/children` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined and validated in `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`

34. Atomic card: REST Client workflow (OpenAPI/Swagger-constrained)
- Endpoints: `POST/PUT/GET /v6/tools/restClients` + optional `POST /v6/tools/copy` + `PUT /v6/suites/children` + `POST /v6/testExecutions`
- Status: Deferred/on hold. Current recommended path is unconstrained REST Client creation via `docs/skills/client-tools/skill-020-rest-client-none-mode-workflow.md`.

35. Atomic card: Create new empty `.tst`
- Endpoint: `POST /v6/files/tsts` (+ optional rollback `DELETE /v6/files?id=...`)
- Status: Defined and validated in `docs/skills/platform/skill-021-tst-create-empty.md`

36. Atomic card: Create `.tst` from OpenAPI/Swagger service definition
- Endpoint: `POST /v6/files/tsts/swagger` (+ optional rollback `DELETE /v6/files?id=...`)
- Status: Defined and validated in `docs/skills/platform/skill-022-tst-create-from-openapi.md`

37. Atomic card: Create `.tst` from WSDL service definition
- Endpoint: `POST /v6/files/tsts/wsdl` (+ optional rollback `DELETE /v6/files?id=...`)
- Status: Defined and validated in `docs/skills/platform/skill-023-tst-create-from-wsdl.md`

38. Atomic card: Create `.tst` from RAML service definition
- Endpoint: `POST /v6/files/tsts/raml` (+ optional rollback `DELETE /v6/files?id=...`)
- Status: Defined (endpoint contract validated) in `docs/skills/platform/skill-024-tst-create-from-raml.md`; runtime validation pending reachable RAML source

39. Atomic card: Create `.tst` from XSD definition
- Endpoint: `POST /v6/files/tsts/xsd` (+ optional rollback `DELETE /v6/files?id=...`)
- Status: Defined and validated in `docs/skills/platform/skill-025-tst-create-from-xsd.md`

40. REST Client constrained creation cards (Skills 026/027)
- Status: Retired from active skill list for now due to unreliable constrained-mode outcomes; use Skill 020 until constrained API behavior is stable.

41. Atomic card: JSON Data Bank extraction workflow (cross-tool)
- Endpoints: `POST/PUT/GET /v6/tools/jsonDataBanks` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined in `docs/skills/data-exchange/skill-028-json-databank-extraction-workflow.md`; runtime validation pending.

42. Atomic card: JSON Validator workflow (cross-tool)
- Endpoints: `POST/PUT/GET /v6/tools/jsonValidators` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined in `docs/skills/validation/skill-029-json-validator-workflow.md`; runtime validation pending.

43. Atomic card: XML Validator workflow (cross-tool)
- Endpoints: `POST/PUT/GET /v6/tools/xmlValidators` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined in `docs/skills/validation/skill-030-xml-validator-workflow.md`; runtime validation pending.

44. Atomic card: Diff Tool workflow with mode matrix (XML/JSON/Text/Binary)
- Endpoints: `POST/PUT/GET /v6/tools/diffTools` + `POST /v6/testExecutions` + `GET /v6/testExecutions/{id}/status` + `GET /v6/testExecutions/{id}/results?includeXmlReport=true`
- Status: Defined in `docs/skills/validation/skill-031-diff-tool-workflow.md`; per-mode ignore-setting validation pending.

45. Cross-cutting card: client header ownership policy
- Scope: define explicit-vs-tool-managed request headers for client tools; default rule is payload-driven `Content-Type` and no manual override.
- Status: Defined in `docs/skills/cross-cutting/skill-032-client-header-ownership.md`.

46. Composite orchestration card: service-test intent intake and plan branching
- Scope: gather user testing scope from underspecified requests, summarize choices, and map confirmed plan to atomic generation/validation/data-exchange skills.
- Status: Defined in `docs/skills/composite-orchestration/skill-033-service-test-intent-orchestration.md`; first live run completed with hardening items captured, rerun pending.

47. Atomic card: subset prune for generated service-definition suites/tests
- Scope: remove non-selected operation suites/tests after full-definition generation so final `.tst` matches user-selected subset intent.
- Status: Defined and validated in `docs/skills/structure/skill-047-generated-subset-prune.md`.

48. Composite orchestration hardening rerun (Skill 033)
- Scope: validate source snapshot capture, fresh-id/parent guards, strict subset pruning, deterministic negative naming, and response-output validator/assertor chaining.
- Status: In progress; rerun executed with source snapshot + subset prune + naming/id guards + response-output JSON validator/assertor chaining.

49. Cross-cutting card: tool PUT read-merge-write update policy
- Scope: require GET -> mutate -> PUT for editing any pre-configured tool (SOAtest/Virtualize) to preserve existing configuration and prevent sparse PUT regressions.
- Status: Defined in `docs/skills/cross-cutting/skill-049-tool-put-read-merge-write-policy.md`.

50. Cross-cutting card: server API capability preflight
- Scope: verify endpoint-method compatibility before orchestration writes and route to documented fallback paths for unsupported methods.
- Status: Defined in `docs/skills/cross-cutting/skill-050-server-api-capability-preflight.md`.

## Virtualize-Specific (Later)

13. Identify and classify `.pva` assets under `/VirtualAssets`
14. Reuse shared lifecycle for `.pva` local edit flow
15. Add deploy-aware upload handling for virtual assets
