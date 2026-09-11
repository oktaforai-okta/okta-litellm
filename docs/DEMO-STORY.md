# Demo story

## What the customer should see

The same Claude Code task is run twice against the Swiss Army MCP. The resource is deterministic and should use synthetic data only.

### Act 1: LiteLLM without O4AA

Claude uses a LiteLLM key and a static or locally issued upstream credential. LiteLLM can still identify its own key, team, user, or local agent and can produce useful logs. The missing controls are specifically Okta-governed controls:

- no O4AA workload principal with owner and lifecycle;
- no Okta Resource Connection governing the target;
- no Okta-issued artifact binding human `sub` to agent `act.sub`;
- no Okta policy/token-grant event to explain the resource authorization;
- no Okta agent deactivation control over fresh work.

Do not say "there is no identity" or "there are no logs." The defensible statement is that there is no Okta-governed agent identity and no Okta-verifiable delegation chain.

### Act 2: Okta PDP + LiteLLM PEP

1. The employee signs in through Okta.
2. Telemetry first proves Claude model inference is routed through LiteLLM's Anthropic-compatible endpoint.
3. Claude invokes one harmless read-only Swiss Army tool through its separate, dedicated LiteLLM MCP route.
4. LiteLLM admits the bound caller and performs the native two-leg ID-JAG exchange.
5. Okta policy permits a token only for the configured resource and read scope.
6. Swiss Army validates the final token and returns synthetic data.
7. A raw call to a simulated write tool shows the precise layer that denies it; do not imply Okta saw the tool name if it only decided the resource scope.
8. The presenter opens an evidence view showing delegated human session, logical Claude route identity, target, scope, tool, outcome, and unambiguous correlated events.
9. The presenter warms the cache, removes eligibility or deactivates the agent, and measures any already-authorized access until expiry without manual eviction.
10. A forced fresh exchange is denied; LiteLLM makes no new protected upstream call.

### Act 3: Why the hybrid remains

Show the architecture slide for GitHub. GitHub requires Okta STS brokered consent/provider OAuth, not the internal Custom-AS XAA token used by Swiss Army. LiteLLM has generic token exchange but not proven parity with O4AA's `oauth-sts` connection and consent workflow. The customer can retain LiteLLM model routing and keep MCP Bridge for that resource until the STS integration is built and verified.

### Act 4: The same governance for a first-party agent

Repeat the safe allow/deny/evidence/lifecycle story through the custom agent UI. Show its distinct O4AA workload principal, owner, credential, Resource Connection, LiteLLM/Bridge binding, and evidence identity. Run it concurrently with Claude and prove that sharing the model router and resource does not share agent credentials, user assertions, token caches, or custody records.

## Primary test personas

| Persona | Expected outcome |
|---|---|
| Allowed employee + active Claude agent | Read succeeds |
| Ineligible employee + active Claude agent | Okta grant denied; no resource action |
| Allowed employee + inactive Claude agent | Fresh grant denied; no new authorized work |
| Allowed employee + wrong LiteLLM key | Gateway admission denied |
| Direct caller + wrong audience/scope token | Swiss Army denies |
| User A key + User B identity token | Denied; no identity substitution |
| Copied Claude route key used by raw client | Denied, or explicitly demonstrates the logical-route attestation limit |
| Valid resource bearer from untrusted host | Network/sender control denies |

## Evidence card for one action

Present one sanitized record containing:

- UTC time and demo correlation ID;
- LiteLLM request ID and hashed key identifier;
- Okta human subject and Claude workload principal;
- Okta grant/deny event identifier where exposed;
- resource audience and effective scopes;
- MCP server and namespaced tool;
- allow/deny/error stage;
- upstream status and side-effect identifier;
- hashes, not raw sensitive arguments or results;
- no access token, ID-JAG, refresh token, secret, or private key.

## Presenter language

Use: "Okta decides whether this delegated human session and customer-controlled logical agent identity can receive a resource-scoped credential. LiteLLM enforces that outcome at the MCP gateway, and the resource independently verifies it. Claude's model traffic is routed through a separate LiteLLM plane."

Avoid: "LiteLLM now has full MCP Bridge parity," "Okta approved this exact tool argument," "Okta attested the Claude binary," "deactivation instantly revokes every cached bearer," or "LiteLLM has no identity without Okta."

## Optional secondary demonstration

After the primary gates pass, add either:

- a simulated write tool with a separate scope and object-level authorization; or
- GitHub through the hybrid MCP Bridge path to explain why XAA and STS are different controls.

Do not make GitHub the first proof: it hides the Okta `act` chain behind provider OAuth and introduces consent, connector, and cloud-resource variables unrelated to the core architecture question.
