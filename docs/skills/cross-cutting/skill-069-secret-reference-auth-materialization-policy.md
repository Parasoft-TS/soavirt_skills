# Skill 069: Secret Reference Consumption + Auth Materialization Policy
## 1) Skill Name
Resolve approved secret references for runtime use and permit plaintext auth materialization in supported SOAtest write branches.
## 2) Objective
Keep one canonical boundary between bootstrap secret storage and later runtime/write-time secret use:
- project/bootstrap workflows store only secret reference information in durable context
- runtime workflows may resolve those approved secret references from gitignored workspace files for direct exploration, execution, or supported SOAtest auth mutation
- supported auth-writing leaves may materialize the resolved credential values into `.tst` assets when that is required to produce the authored test asset the user asked for
- protecting the generated asset before source-control commit remains the user's responsibility unless a workflow explicitly says otherwise
## 3) Scope
- In scope:
  - interpreting approved secret-reference paths loaded from project context
  - resolving plaintext secret values from gitignored workspace files for live runtime branches
  - distinguishing secret-reference storage from later runtime consumption
  - permitting plaintext/native-auth materialization for supported SOAtest auth mutation branches
  - blocked-auth classification rules when a secret reference exists but cannot be used
  - preventing environment/reference-indirection workarounds from becoming the default concealment strategy
- Out of scope:
  - bootstrap secret-storage prompts and project-record persistence mechanics (owned by Skill 063)
  - inventing credentials, secrets, or tokens
  - unsupported auth-scheme design or future secret-protection automation for tracked `.tst` files
  - forcing external environment references merely to avoid plaintext auth materialization in a supported write branch
## 3.1) Canonical Ownership Boundaries
- Skill 063 owns durable secret-reference storage and project-record path references.
- This card owns the downstream interpretation of those approved references for runtime use.
- Skills 020 and 059 own the actual REST Client auth mutation mechanics.
- Skills 065 and 066 own how exploration-first runtime branches consume the policy.
## 4) Inputs
- Required:
  - a runtime branch that actually needs credentials
  - explicit user-provided credentials or an approved secret reference already available from project context/current session
- Optional:
  - the target supported auth surface (for example REST Client Basic auth)
  - explicit user direction requesting a different secret-handling strategy
## 5) Preconditions
- The secret source is explicit user input or an approved gitignored secret reference; do not guess secrets.
- The owning mutation/execution branch already has permission to act on the target asset.
## 6) Procedure
1. When a runtime branch needs credentials, resolve them from this precedence:
   - explicit current-task user input
   - approved secret reference already loaded from project context/current session
2. If the source is an approved secret reference:
   - resolve the plaintext values from the referenced gitignored workspace file
   - do not duplicate those values into the durable project record merely because they were consumed
3. Use the resolved credentials directly for live runtime branches that need them:
   - direct exploration
   - focused verification/execution
   - supported SOAtest auth mutation
4. For supported SOAtest auth-write surfaces, plaintext/native materialization is valid:
   - write the resolved values into the owning leaf's native auth subtree
   - do not classify the branch as `blocked-auth` merely because the resulting `.tst` will now contain the credential values
5. Do not open a referenced-environment or other concealment workaround solely to avoid persisting supported auth values into the authored `.tst` unless the user explicitly asks for that strategy.
6. If the approved secret reference is unreadable, missing, incomplete, or incompatible with the supported auth path, classify the branch as truly blocked and say why.
7. Treat post-generation source-control hygiene as a separate user responsibility unless an owning workflow explicitly introduces a stronger safeguard.
## 7) Validation
- Bootstrap/project context stores only secret references, not duplicated secret values.
- Runtime branches may consume approved secret references without reopening secret-storage questions.
- Supported auth-write leaves may materialize resolved credentials directly into `.tst` auth config.
- `blocked-auth` is reserved for genuinely unusable auth input, not for discomfort with plaintext auth materialization.
## 8) Failure Modes
- A bootstrap storage rule is over-applied and later runtime branches refuse to use an approved secret reference.
- The agent invents or guesses credentials instead of using explicit input or approved references.
- The agent opens an environment/reference-indirection branch purely to avoid supported plaintext auth materialization.
- Secret values are copied back into durable project metadata just because they were consumed at runtime.
## 9) Safety / Rollback
- Keep durable project context reference-oriented.
- Keep secret-resolution side effects limited to the runtime branch that actually needs them.
- Leave any stronger post-write secret-protection or source-control cleanup to explicit user direction or a future dedicated workflow.
## 10) Reuse Notes
- Use this card whenever contributor or runtime work needs to separate secret-reference storage from later auth materialization.
- This card does not replace the auth mechanics in Skills 020/059; it clarifies when those mechanics may consume approved secret references and write the resulting values into `.tst` assets.
