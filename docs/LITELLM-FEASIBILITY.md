# LiteLLM feasibility assessment

Assessment date: 2026-09-11.

## Verdict

LiteLLM can be the primary MCP gateway PEP for a narrow internal XAA demo. It is not currently a drop-in replacement for every MCP Bridge function.

The native path is strongest when all calls on one MCP route represent one O4AA agent and the upstream resource accepts an Okta Custom-AS token. That is exactly why Swiss Army MCP is the recommended first target. Claude and the custom UI therefore require separate configurations/routes and WLP credentials; this demonstrates static isolation, not dynamic fleet-scale identity selection.

## Verified stable capability

| Capability | Status on `v1.100.1` | Relevance |
|---|---|---|
| MCP gateway and single-server route | Available | Claude can use LiteLLM as its MCP endpoint |
| MCP server/tool access controls | Available | Gateway-side enforcement ceiling |
| `oauth2_id_jag` egress | Stable since `v1.94.0` | Native two-leg Okta XAA flow |
| Private-key JWT client authentication | Available | Preferred O4AA agent authentication |
| Generic OIDC assertion storage | Stable by `v1.96.0` | Allows a user-bound virtual key to perform ID-JAG |
| Per-user exchanged-token cache | Available | Performance, with revocation caveats |
| Model routing, budgets, guardrails, logging | Available | Preserves the customer's LiteLLM value |
| Self-signed MCP JWT mode | Available | Not a substitute for an Okta-issued chain |

Release evidence:

- [`v1.100.1`](https://github.com/BerriAI/litellm/releases/tag/v1.100.1) was the latest stable release observed on 2026-09-11.
- ID-JAG MCP egress landed in [commit `e189666`](https://github.com/BerriAI/litellm/commit/e18966625d63847a8c2e476767734bb711a2b88b) and appeared in [`v1.94.0`](https://docs.litellm.ai/release_notes/v1.94.0/v1-94-0).
- Stored SSO ID-token support is documented in [`v1.96.0`](https://docs.litellm.ai/release_notes/v1.96.0/v1-96-0).
- Configuration and current known limitations are in LiteLLM's [ID-JAG guide](https://docs.litellm.ai/docs/mcp_id_jag).

## Exact native exchange

Leg one sends the human Org-AS ID token to the Okta Org-AS `/oauth2/v1/token` endpoint with RFC 8693 token exchange, requests an ID-JAG, and authenticates the configured agent/gateway client. Leg two sends the ID-JAG to the resource Custom-AS token endpoint using the RFC 7523 JWT bearer grant. LiteLLM forwards the returned resource Bearer to the MCP server.

With a configured private key, LiteLLM currently signs its client assertion with `iss=sub=client_id` and the token endpoint as `aud`. The O4AA reference normally identifies the signing principal as the workload principal (`wlp...`). Tenant behavior must decide whether the configured LiteLLM field can hold that principal identifier. This is a must-prove gate, not a naming assumption.

## Important limitations

### Fixed egress identity

The ID-JAG credential belongs to the MCP server configuration, not dynamically to the calling LiteLLM `agent_id`. If multiple logical agents share a route, they share the configured Okta actor. The MVP therefore dedicates the route/WLP to Claude-via-LiteLLM. Fleet-scale support requires per-request validated agent-to-WLP credential selection.

### Human assertion capture

The stored-assertion path depends on LiteLLM's **generic OIDC SSO**. Provider-specific Google/Microsoft and SAML paths do not provide the same ID-token capture for this feature. Stable `v1.100.1` does not contain the later stored-assertion refresh work observed in main, so the demo must include a predictable re-login step after assertion expiry.

On the inspected stable resolver path, an inbound identity bearer may take precedence over the stored assertion without proving its subject matches the LiteLLM key owner. Therefore the primary demo uses stored-assertion-only resolution. Direct-token mode remains disabled unless the exact build passes the cross-user substitution test.

### Discovery/call mismatch

LiteLLM documents a case where an inbound user `Authorization` token can be used for `tools/call` while `tools/list` still depends on a stored assertion. A direct-token user with no stored assertion may see an empty list and a precondition warning. Use one generic-OIDC bootstrap login and verify list/call subject parity on the exact build.

### Two policy planes

Okta decides resource/scope token issuance. LiteLLM independently filters servers/tools through its key, team, user, agent, and organization permissions. Native LiteLLM does not turn all of those local decisions into Okta decisions. Treat local rules as deny-only ceilings and test configuration drift.

There are also two traffic planes: Claude model inference and Claude MCP JSON-RPC. Model routing is configured through LiteLLM's Anthropic-compatible endpoint; MCP routing uses the separate MCP endpoint. Both must be configured and observed independently.

### Tool granularity

ID-JAG scopes are configured at the MCP-server level, while strict authorization may need action/tool-specific scopes. The initial demo requests one read-only scope for one safe route. A later integration must map tools to effective scopes and make discovery and invocation use one decision contract.

### Experimental surface and edition

The MCP implementation resides under LiteLLM's `_experimental` package in the inspected source. Some adjacent ingress/JWT/SSO features have edition checks. Pinning, feature/edition validation, and regression tests are mandatory.

### Cache behavior

The inspected stable exchanged-token cache is process-local and may fall back to a one-hour cache when leg-two `expires_in` is absent or unusable. The demo must require a valid short lifetime, measure effective cache age, and treat restart or natural expiry as the only eviction methods until a supported eviction API is proven. Deactivation claims apply to fresh issuance, not already-issued bearers.

### What native LiteLLM does not replace

- O4AA agent and Resource Connection onboarding/synchronization;
- Okta OPA vaulted-secret exchange;
- proven Okta `oauth-sts` brokered-consent behavior and interaction flow;
- automatic mapping of LiteLLM agents to separate O4AA WLP credentials;
- MCP Bridge's established O4AA-specific evidence and lifecycle behavior.

## Stable versus prerelease policy

Use stable `v1.100.1` for the first spike. `v1.101.0-rc.1/rc.2` and `v1.102.0-dev.*` were prereleases as of the assessment date and contain useful ID-JAG preflight, assertion renewal, and warning improvements. Do not base the customer claim on them. When those changes ship in a stable release, repeat the entire validation matrix before changing the pin.

## Extension seams if the MVP passes

The inspected source provides credible integration points:

- custom ingress authentication can map validated Okta claims into LiteLLM user/agent context;
- `pre_mcp_call` sees server, tool, arguments, user/team/end-user, key hash, and sanitized headers;
- the typed outbound credential resolver already owns ID-JAG and fail-closed token injection;
- a narrow `tools/list` hook is needed for a shared dynamic decision contract;
- structured/custom logging callbacks can emit a custody receipt.

Do not use the LiteLLM MCP JWT signer as the O4AA integration. It self-signs and currently derives `act.sub` from team/organization context rather than the O4AA agent.

## Decision

Proceed only through the gates. A passing native demo validates LiteLLM as PEP for one O4AA XAA resource. It does not validate GitHub STS, OPA, or multi-agent parity. If the actor claim or lifecycle path fails, choose the thin integration or hybrid architecture instead of weakening the identity requirement.
