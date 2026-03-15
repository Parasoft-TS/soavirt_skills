# Skill 032: Client Header Ownership (Cross-Tool)

## 1) Skill Name
Client header ownership and payload-driven content type behavior

## 2) Objective
Define which request headers should be explicitly authored in client tools versus which headers or auth settings should be owned by the client tool's native configuration model.

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
- For REST Clients, when the requested auth scheme is supported by the native REST Client auth model, prefer `httpOptions.transport.<http10|http11>.security.httpAuthentication.performAuthentication` over explicit `Authorization` header injection.

## 5) What To Set Explicitly
- Commonly explicit:
  - `Accept`
  - endpoint-specific custom headers required by the target service
- Conditionally explicit:
  - `Authorization` only when the native client auth model does not support the intended scheme or the user explicitly wants literal custom-header behavior
- Usually not explicit:
  - `Content-Type` when payload configuration controls body encoding/content type
  - `Authorization` for REST Client Basic auth, because the native auth subtree is the preferred owner

## 6) Why This Exists
- Reduces drift between payload configuration and wire-level headers.
- Avoids conflicts where manually forced header values diverge from tool-managed payload semantics.
- Improves portability of the same skill pattern across client tools.

## 7) Procedure Checkpoints
1. Configure payload/body first (mode + content type).
2. If auth is needed, determine whether the client has a native auth model for the requested scheme before adding any literal `Authorization` header.
3. Add only non-owned headers explicitly (`Accept`, custom service headers, and literal `Authorization` only for unsupported/custom auth schemes).
4. Read back tool config and verify explicit header list does not include unnecessary `Content-Type` overrides or a literal `Authorization` header when the native REST Client auth model owns auth.
5. Validate with a focused run and inspect the effective request/auth behavior in traffic evidence.

## 8) Failure Modes
- Header drift: manually set `Content-Type` conflicts with payload mode/content type.
- Auth-ownership drift: a supported REST Client auth scheme is forced through literal `Authorization` headers instead of the native auth subtree.
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
- REST Client Basic-auth preference for the native auth subtree over literal `Authorization` header injection is validated in the current workspace.
- Future explicit runtime validations should be appended as client-tool cards are added.
