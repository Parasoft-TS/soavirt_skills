# Skill 054: XPath Scalar Extraction Normalization (Cross-Cutting)

## 1) Skill Name
XPath scalar extraction normalization for databanks and assertors

## 2) Objective
Standardize XPath generation for scalar-value extraction/assertion so XML tools avoid element-markup propagation (for example `<id>123</id>`), while preserving JSON XPath-over-JSON behavior.

## 3) Scope
- In scope:
  - XML Data Bank selector generation
  - XML Assertor selector generation
  - scalar extraction/assertion selector normalization for tools that use XPath
  - boundary guidance for JSON selector fields
- Out of scope:
  - media-type selection (owned by skill families + Skill 017/018)
  - non-scalar node extraction intent (entire element/subtree use cases)

## 4) Inputs
- Required:
  - observed runtime media type (XML vs JSON)
  - selector intent (scalar value vs node/structure)
  - candidate XPath
- Optional:
  - extraction type/options (`contentOnly`, `entireElement`)
  - downstream consumer type (URL path/query/header/assertion expected value)

## 5) Preconditions
- Runtime payload type is known from baseline evidence.
- Selector field is XPath-based.

## 6) Procedure
1. Determine media type and selector intent:
  - XML + scalar intent -> apply normalization.
  - JSON (XPath-over-JSON) -> keep selector as authored unless tool-specific behavior requires otherwise.
2. XML scalar normalization rule:
  - if XPath targets an element and scalar text is intended, append `/text()` to the terminal selector.
3. Do not append `/text()` when:
  - XPath already ends in `text()`,
  - selector is attribute-based (for example `.../@id`),
  - selector intentionally returns a node/subtree (for example entire-element extraction intent).
4. Preserve idempotence:
  - applying normalization repeatedly must not change XPath after first normalization.
5. Validate with readback and one focused execution when selector is used by downstream URL/header/assertion logic.

## 7) Validation
- XML scalar selectors resolve to plain scalar values without embedded element markup.
- URL/path/query substitutions fed by databank columns produce valid request targets.
- Assertions compare scalar values as intended.
- JSON selector behavior remains unchanged.

## 8) Failure Modes
- Missing `/text()` on XML scalar intent can propagate serialized element markup into downstream variables.
- Over-appending `/text()` to non-scalar intent can remove required structure.
- Applying XML normalization rules to JSON selectors can introduce unnecessary churn.

## 9) Safety / Rollback
- This is a policy/authoring skill (no direct write endpoint ownership).
- Rollback by restoring previous selector values on affected tools.

## 10) Reuse Notes
- Primary target: SOAtest.
- Applicable in Virtualize.
- Intended as a dependency for XPath-producing databank/assertor skills.
