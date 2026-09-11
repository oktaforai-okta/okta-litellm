# ADR 0001: Evaluate LiteLLM as the primary MCP gateway PEP

- Status: Proposed
- Date: 2026-09-11

## Context

The customer values LiteLLM's model routing and already operates it. Okta should remain the agent identity and policy decision plane. MCP Bridge is the most complete Okta-native enforcement option, but requiring it for every MCP request creates another gateway for this customer.

LiteLLM stable supports MCP routing, local server/tool controls, generic OIDC identity assertion storage, and native Okta ID-JAG egress. Its O4AA actor mapping, resource synchronization, STS/OPA parity, and per-tool dynamic Okta decision behavior are incomplete.

## Decision

Run a gated LiteLLM-first MVP for one dedicated Claude-via-LiteLLM workload principal and one internal XAA-protected Swiss Army MCP route. After that path passes, repeat it for a separately registered custom agent UI using a distinct WLP, credential, and route. Okta decides whether to issue each user-plus-agent resource credential; LiteLLM enforces admission, route/tool ceilings, token exchange outcome, and forwarding; Swiss Army validates and enforces the resource token.

Do not remove the hybrid option or claim general MCP Bridge replacement from this result.

## Consequences

Positive:

- Preserves the customer's LiteLLM deployment and model-routing value.
- Tests the most direct native integration with little initial core customization.
- Produces evidence about the actual WLP/client assertion compatibility.

Negative:

- One configured egress identity requires a dedicated route/instance per logical MVP agent.
- Strict list-time and tool-specific Okta decisions likely require a thin integration.
- OPA, STS, synchronization, and mature O4AA operational workflows remain outside the MVP.
- The source surface is experimental and must be pinned and regression-tested.

## Reversal criteria

Adopt the hybrid architecture if the final token does not identify the intended Claude workload principal, fresh authorization cannot be blocked predictably, resource-side enforcement cannot be proven, or the required LiteLLM edition/source changes exceed the agreed demo scope.
