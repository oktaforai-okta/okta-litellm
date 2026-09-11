# ADR 0003: Retain LiteLLM plus MCP Bridge as the reference and fallback

- Status: Proposed
- Date: 2026-09-11

## Context

MCP Bridge already covers more O4AA-specific responsibilities: agent/resource mapping, XAA, brokered-consent STS, vaulted secrets, synchronization, and established custody behavior. LiteLLM is strongest in model/provider routing and has a promising but narrower MCP identity integration.

## Decision

Maintain a supported architecture in which LiteLLM routes model traffic and MCP Bridge remains the MCP gateway/PEP. It is both the control arm for feature comparison and the fallback when a resource needs capabilities outside the proven LiteLLM path.

Fallback is an explicit deployment decision. A LiteLLM deny, Okta outage, or token error must never cause an individual request to route automatically through MCP Bridge or directly to the resource.

The executable topology keeps Claude model inference on LiteLLM and changes only Claude's MCP endpoint to a pre-provisioned MCP Bridge. The switch is manual, clears incompatible sessions/caches, requires fresh Bridge authentication, verifies one allow and one denial, and is rehearsed against an approved recovery-time target.

## Consequences

- Lower identity and credential-brokering risk for GitHub and other STS/OPA resources.
- Faster path to a customer-safe mixed environment.
- Additional endpoint, operational, and evidence correlation complexity.
- Creates a staged migration path instead of an all-or-nothing replacement.
- Requires pre-provisioning, named operational ownership, periodic rehearsal, and evidence normalization across two gateways.
