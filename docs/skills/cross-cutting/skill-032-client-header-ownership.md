# Skill 032: Client Header Ownership (Cross-Tool)

## 1) Skill Name
Client header ownership and payload-driven content type behavior

## 2) Objective
Define which request headers should be explicitly authored in client tools versus which headers should be emitted automatically by the client tool based on payload configuration.

## 3) Scope
- In scope:
  - REST Client header ownership defaults
  - forward rule for SOAP Client, GraphQL Client, and Messaging Client skills
  - verification checkpoints for readback and runtime request headers
- Out of scope:
  - endpoint-specific business rules
  - auth scheme design (for example OAuth token generation)

## 4) Core Rule
- For SOAtest client tools, do **not** manually set `Content-Type` request header when payload configuration already defines content type.
- Set payload content type in tool payload fields (for example `payload.contentType`) and let the client tool emit `Content-Type` automatically.

## 5) What To Set Explicitly
- Commonly explicit:
  - `Accept`
  - `Authorization`
  - endpoint-specific custom headers required by the target service
- Usually not explicit:
  - `Content-Type` when payload configuration controls body encoding/content type

## 6) Why This Exists
- Reduces drift between payload configuration and wire-level headers.
- Avoids conflicts where manually forced header values diverge from tool-managed payload semantics.
- Improves portability of the same skill pattern across client tools.

## 7) Procedure Checkpoints
1. Configure payload/body first (mode + content type).
2. Add only non-owned headers explicitly (`Accept`, `Authorization`, custom service headers).
3. Read back tool config and verify explicit header list does not include unnecessary `Content-Type` overrides.
4. Validate with a focused run and inspect request header in traffic evidence.

## 8) Failure Modes
- Header drift: manually set `Content-Type` conflicts with payload mode/content type.
- False debugging path: endpoint failures misattributed to payload schema when actual issue is header override behavior.
- Portability risk: copied tool config carries unnecessary manual header overrides into other client tools.

## 9) Reuse Notes
- REST Client: apply immediately.
- SOAP Client: apply when SOAP skill card is authored.
- GraphQL Client: apply when GraphQL skill card is authored.
- Messaging Client: apply when Messaging skill card is authored.
- This card is cross-cutting policy and should be referenced by all client-tool lifecycle cards.

## 10) Validation Status
- Policy validated in current workspace through REST Client creation/refinement flow.
- Future explicit runtime validations should be appended as client-tool cards are added.
