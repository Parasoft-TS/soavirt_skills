# Skill 011: XPath-over-JSON Query Semantics (Cross-Cutting)

## 1) Skill Name
XPath-over-JSON query semantics for SOAtest/Virtualize

## 2) Objective
Apply XPath-style expressions (not JSONPath) when selecting data in tools that operate on JSON payloads, because SOAtest/Virtualize use a unified query engine across XML and JSON.

## 3) Scope
- In scope:
  - JSON Assertor selected-element paths
  - JSON Data Bank selected-element paths
  - Any tool/config field expecting an XPath-like selector, even when payload format is JSON
- Out of scope:
  - Native JSONPath syntax authoring
  - Non-selector fields unrelated to element targeting

## 4) Core Rule
When a SOAtest/Virtualize field expects an XPath-type expression, provide XPath-style selectors for both XML and JSON payloads.

## 5) Why This Exists
SOAtest/Virtualize use a custom interoperability engine that maps XPath-style selection semantics onto JSON structures, enabling a common query language across XML and JSON workflows.

## 6) Practical Guidance
- Prefer selectors like:
  - `/root/item`
  - `/root/item/customerId`
  - `/root/item/type`
  - `/root/listItem[3]/id`
  - `/root/listItem[${variable}]/name`
- Do not switch to JSONPath in these selector fields unless a field explicitly documents JSONPath support.
- Keep selector style consistent across assertors/data banks to reduce mixed-syntax errors.
- SOAtest/Virtualize support variables in XPaths, allowing to build complex selector logic that references dynamic data that comes from other places.

## 7) Validation Checklist
1. Confirm tool field is an XPath-style selector field.
2. Save configuration with XPath expression.
3. Read back tool config and ensure selector persisted in XPath form.
4. Validate assertions/data extraction against a known JSON sample payload.

## 8) Failure Modes
- Using JSONPath syntax in XPath-expected fields causes parse/runtime failures or incorrect selection.
- Mixed XPath/JSONPath conventions across tools lead to brittle test suites.

## 9) Reuse Notes
- Applies to SOAtest: Yes.
- Applies to Virtualize: Expected by shared engine behavior; validate per tool when first encountered.
- Should be loaded as a dependency whenever configuring selector expressions on JSON payload tools.
