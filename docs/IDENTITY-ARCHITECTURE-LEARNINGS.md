# Identity architecture learnings from Okta's gateway blog

## Source and evidence status

Source: [Your AI gateway has an identity crisis: Fixing the last-mile problem in enterprise AI](https://www.okta.com/blog/ai/securing-ai-agents-identity-architecture/), Alex Kemenov, Okta, published 2026-09-01.

Reviewed on 2026-09-11. This is an official Okta architecture and positioning article, not a versioned product API contract. Use it to explain the problem, security model, and customer value. Use Okta Help, API documentation, tenant behavior, and pinned implementation evidence for build-specific claims.

## The new architecture lens

The article gives this project a clear name for the customer problem: the **last-mile identity problem**.

A gateway is valuable for model routing, cost controls, rate limits, tool discovery, admission, and operational telemetry. A gateway credential by itself does not necessarily give the downstream resource a live, verifiable answer to all of these questions:

1. Which human initiated this request?
2. Which governed agent is acting for that human?
3. Is that agent allowed to act for this human?
4. Is that agent allowed to reach this resource?
5. What is this human currently entitled to do at the resource?

This distinction fits the customer's requirement to keep LiteLLM. The solution is not to remove a useful gateway. It is to add a governed identity and authorization layer whose result travels through the gateway to the protected resource.

## Required authorization intersection

The effective authority for a protected call is the intersection of three independent ceilings:

```text
effective authority
  = agent may act for this human
  ∩ agent may reach this resource
  ∩ human is entitled to this resource action
```

In the repository's primary XAA design:

| Question | Governing evidence/control |
|---|---|
| May this agent act for this human? | Okta agent/delegation policy at the Org Authorization Server |
| May this agent reach this resource? | Active O4AA workload principal, Resource Connection, target, and allowed scopes |
| What may this human do there? | Resource Custom Authorization Server policy plus resource-side scope/object enforcement |
| What reaches the MCP server? | Short-lived audience-bound resource token carrying the tenant-issued human/agent authorization result |

Neither a gateway allowlist nor a user-only token is the complete intersection. LiteLLM must preserve the verified user binding, select only the intended workload credential, request the correct resource/scope, stop on denial, and forward only the resulting resource token.

## Why a static or per-user gateway key is insufficient

The blog uses static keys to show the failure mode. This repository keeps the claim precise:

- A LiteLLM key can identify an internal user, team, budget, route, or configured logical agent and can support useful controls and logs.
- A key alone does not prove the holder's current Okta entitlement to the downstream resource.
- A shared or copied key can collapse the human and agent identities.
- A per-user key is still not a substitute for a short-lived, audience-bound Okta resource token evaluated at request time.
- A downstream MCP server must not trust user, agent, scope, or approval values supplied as tool arguments or arbitrary headers.

The control demonstration must therefore show exactly which Okta-governed properties are missing. It must not inaccurately claim that LiteLLM has no identity or no logs.

## Why user-only on-behalf-of is incomplete

Adding human login is useful, but simply borrowing or forwarding the user's authority can erase the acting agent. That leaves a confused-deputy risk and prevents agent-specific ownership, reachability, policy, credential lifecycle, and deactivation.

The governed path must preserve two distinct principals:

- the human subject whose entitlement ceiling applies; and
- the workload principal for the logical agent performing the action.

The build must reject a flow that produces only a human identity when the customer claim requires human-plus-agent delegation.

## Two authorization points, one resource result

The article's two-authorization-server explanation maps directly to the planned native LiteLLM XAA flow:

1. The Okta Org Authorization Server evaluates the agent/delegation side and issues the intermediate ID-JAG or denies it.
2. The resource's Custom Authorization Server evaluates the user/resource side and issues the scoped resource bearer or denies it.
3. The MCP server validates the final token and enforces the resulting scope and any object-level rule.

Protocol wording matters. The intermediate ID-JAG is short-lived and single-use. The final resource access token is short-lived and scoped, but must not be described as single-use unless the deployed tenant/resource proves that property.

The resource does not need to duplicate central identity policy, but it must validate the token on every protected call and enforce scopes, claims, tool arguments, and business objects as required.

## Consequences for the LiteLLM-first design

| Blog principle | Repository consequence |
|---|---|
| Keep the gateway for the job it does well | Retain LiteLLM model routing, budgets, rate limits, admission, MCP aggregation, and telemetry |
| Identity must survive the last mile | Final MCP request carries the Okta resource token, not only the LiteLLM key |
| Agent is a first-class identity | Separate O4AA WLP, owner, credential, Resource Connection, and lifecycle for Claude and custom UI |
| Authorization is request-level | Resolve the human binding and perform/validate fresh token issuance at the protected request boundary |
| Human authority is a ceiling | Agent/gateway permissions may narrow but never widen the user's current resource entitlement |
| Resource receives the verdict | Swiss Army validates issuer, signature, audience, time, scope, and tenant-issued actor profile |
| Governance is centralized | Okta remains source of truth for identity, delegation, connection, policy, and lifecycle |
| Developer workflow should remain natural | One normal login/consent experience; token mechanics remain behind the client unless action is required |

## Consequences for the hybrid design

The same principles apply when MCP Bridge remains the MCP PEP:

- LiteLLM continues to route model traffic.
- MCP Bridge performs inbound MCP OAuth, deterministic agent selection, Resource Connection resolution, and XAA/STS orchestration.
- Okta remains the PDP and governed identity plane.
- The downstream resource/provider remains the final token enforcement point.

This is complementary architecture, not a claim that an identity layer replaces every gateway function.

## Demo changes

The demo must make the authorization ceiling visible rather than merely showing a successful token exchange.

### Control path

Show that LiteLLM can route the model/tool request, enforce its local key and route configuration, and log the call. Then identify the missing Okta-governed facts:

- no governed agent owner/lifecycle;
- no agent-specific Resource Connection;
- no Okta-verifiable human-plus-agent delegation artifact at the resource;
- no Okta request-time token decision explaining the resource access; and
- no agent deactivation control over fresh authorization.

### Governed path

Using the same logical task and MCP resource:

1. an eligible human with the active agent succeeds;
2. an ineligible human with the same agent is denied;
3. the eligible human with an inactive/unreachable agent is denied on a fresh exchange;
4. the resource rejects a LiteLLM key or wrong-audience token presented directly; and
5. evidence correlates the human, agent, Resource Connection, requested resource/scope, Okta result, gateway action, and downstream outcome.

If the resource supports safe object-level authorization, add a synthetic ceiling example: the same user and agent may access the user's own object but not another user's object. This is optional and requires resource-side enforcement or FGA; OAuth scope alone must not be presented as object authorization.

### Developer-experience requirement

The normal allowed flow should not require the developer to understand ID-JAG or manually move tokens. Measure:

- initial login steps;
- reauthentication/consent prompts;
- time added to first and warm calls; and
- clarity of denial and remediation.

Security must remain visible to administrators and auditors without turning ordinary developer work into a token-handling exercise.

## Additional validation requirements

The build is not complete until it proves:

- a gateway key alone cannot authorize the protected MCP resource;
- a human-only identity cannot silently satisfy the agent-identity claim;
- the same agent/resource request produces different results for eligible and ineligible humans;
- an eligible human cannot exceed their entitlement through a more privileged agent or gateway route;
- an active user cannot use an inactive or disconnected agent for fresh authorization;
- a caller cannot substitute a different agent, user, route, resource, or scope;
- the final resource token is the intersection produced by the intended tenant flow;
- the resource independently validates and enforces that token; and
- allowed and denied paths preserve a correlated, sanitized evidence trail.

## Customer language

Use:

> LiteLLM remains the customer's gateway for model routing and operational controls. Okta supplies the governed human-plus-agent identity and request-time authorization ceiling. LiteLLM or MCP Bridge enforces the token outcome inline, and the protected resource validates the scoped token before acting.

Use:

> A gateway key can identify a configured caller or route, but the Okta resource token carries the current intersection of the governed agent and human entitlement to the resource.

Avoid:

- “LiteLLM has no identity or logging.”
- “Every gateway is incapable of carrying user context.”
- “A per-user API key is anonymous.”
- “Okta approves every tool argument.”
- “The resource no longer needs authorization checks.”
- “The final bearer is single-use” unless proven.
- “Agent deactivation instantly invalidates every cached token.”

## Builder decision rule

For every protected MCP path, the builder must be able to point to:

1. the human identity source;
2. the governed agent/WLP and owner;
3. the agent-to-resource connection;
4. the two authorization decisions or release-equivalent provider decision;
5. the final resource audience and scope;
6. the inline gateway enforcement step;
7. the downstream enforcement step; and
8. the correlated allow/deny evidence.

If any answer is only “the gateway key allowed it,” the last-mile identity problem remains unsolved.
