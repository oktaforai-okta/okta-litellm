# Claude Code and custom agent UI experiences

The build must demonstrate Okta's value through both a third-party client and a first-party client. The clients may share a protected MCP resource, but they must not share logical-agent identities, admission credentials, sessions, or token caches.

## The Okta value to make visible

| Customer question | What the demonstration must show |
|---|---|
| Which agent is this? | A named O4AA workload principal with owner, purpose, status, credential metadata, and separate identity for each client experience |
| Who is it acting for? | The human subject/delegated session and logical agent actor in the approved token profile and evidence |
| What may it access? | An active Resource Connection with explicit target, audience/resource indicator, and least-privilege scopes |
| Who decided? | Okta policy permits or denies fresh resource-token issuance; LiteLLM or Bridge enforces the outcome and the resource independently validates it |
| What happened and why? | Correlated Okta, gateway, and resource events showing policy/config revision, sanitized task/reason, tool, result, and side-effect ID |
| How is access stopped? | Agent/user/connection lifecycle change prevents fresh authorized work after the measured bearer/cache window |
| What risk is removed? | The client never receives a downstream credential; model routing cannot silently become authorization; invalid audience/scope and bypass attempts fail |

Present Okta as the governed identity and policy/token plane, not as the model router or the tool implementation.

## Experience 1: Claude Code

Claude Code proves that a customer can govern a third-party MCP-capable agent without giving it a downstream static credential.

### Required sequence

1. Show the `Claude-via-LiteLLM` or `Claude-via-MCP-Bridge` O4AA record, owner, active credential, linked user sign-on/client configuration, and Resource Connection.
2. Show Claude's model request traversing LiteLLM's model endpoint.
3. Authenticate the user to the selected MCP gateway through the approved flow.
4. Run one deterministic read tool against synthetic data.
5. Run one explicitly denied operation and show the exact denying layer.
6. Open one sanitized evidence card that joins the human session, logical Claude identity, policy/config revision, target, scope, tool, and result.
7. Change user eligibility, agent status, or Resource Connection state; measure existing bearer/cache behavior and then prove fresh issuance is denied.
8. Show that no direct MCP or model-provider bypass silently replaced the selected route.

For the hybrid topology, Claude's inbound Bridge mode and deterministic agent selection must be verified against the deployed Bridge release. For the LiteLLM-first topology, the stored assertion, LiteLLM user, admission credential, dedicated route, and WLP credential must pass the binding tests.

## Experience 2: Custom agent UI

The custom UI proves the same Okta controls apply to a customer-built first-party experience rather than only to a coding assistant.

### Required user-facing state

Display safe, human-readable metadata:

- signed-in user display name and immutable-subject reference in a diagnostic view;
- custom agent name, owner, purpose, and logical identity reference;
- selected resource and requested business action;
- pending/allowed/denied/error state, identifying whether admission, Okta token issuance, gateway, or resource enforcement produced it;
- correlation ID and link/reference to sanitized evidence;
- reauthentication or downstream-consent action when legitimately required.

Never display raw ID/access tokens, ID-JAGs, client assertions, JWKs, model-provider keys, Bridge admin credentials, or sensitive prompts/tool results. Never label an LLM-generated explanation as the Okta policy reason.

### Required sequence

1. Register a separate custom-UI agent/WLP and owner; do not reuse the Claude identity.
2. Use authorization code with PKCE for a browser public client and an approved backend-for-frontend when secrets or server-side exchanges are required.
3. Route model inference through LiteLLM and protected tools through the selected LiteLLM-first or Bridge-hybrid MCP plane.
4. Repeat the same allow, deny, evidence, lifecycle, and bypass tests used for Claude.
5. Run Claude and the custom UI concurrently and prove there is no credential, assertion, cache, tool result, or evidence crossover.

## Topology acceptance matrix

| Capability | LiteLLM-first | Hybrid |
|---|---|---|
| Model routing | LiteLLM | LiteLLM |
| MCP gateway PEP | LiteLLM | MCP Bridge |
| Identity/token PDP | Okta/O4AA | Okta/O4AA |
| Final resource PEP | Protected MCP server | Protected MCP server or provider |
| Best first resource | Swiss Army MCP with Custom-AS XAA | Swiss Army XAA; GitHub/Atlassian only after STS consent is verified |
| Stop condition | Actor/binding/custody/resource/lifecycle gate fails | Bridge version/client mode/resource linkage/STS-XAA gate fails |
| What it proves | Narrow native LiteLLM enforcement for one dedicated logical agent/resource | Okta-native Bridge enforcement while preserving LiteLLM model routing |

## Before-and-after comparison

The “without Okta” control may still have LiteLLM users, keys, budgets, guardrails, and logs. State the delta precisely:

- Before: no Okta-governed workload principal/owner/lifecycle, no O4AA Resource Connection, no Okta-issued human-plus-agent delegation artifact, no Okta policy-grant event, and no O4AA lifecycle control over fresh authorization.
- After: those Okta controls are visible and verified at issuance and resource enforcement, while LiteLLM continues to provide model routing and gateway functions.

Do not say “there was no identity,” “there were no logs,” “Okta approved every tool argument,” “deactivation revoked every bearer instantly,” or “the Claude binary was attested” unless additional controls actually prove those claims.

## Demo exit criteria

Both experiences must pass identity isolation, positive/negative authorization, direct-bypass, chain-of-custody, secret handling, concurrency, and measured lifecycle gates on the selected topology. A polished UI or visible tool list is not a substitute for those results.

