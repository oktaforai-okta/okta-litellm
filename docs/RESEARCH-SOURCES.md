# Research sources

Reviewed on 2026-09-11.

## Primary sources

- [LiteLLM repository](https://github.com/BerriAI/litellm)
- [LiteLLM documentation](https://docs.litellm.ai/docs)
- [MCP overview](https://docs.litellm.ai/docs/mcp)
- [MCP authentication](https://docs.litellm.ai/docs/mcp_authentication)
- [MCP access control](https://docs.litellm.ai/docs/mcp_control)
- [Okta ID-JAG MCP egress](https://docs.litellm.ai/docs/mcp_id_jag)
- [LiteLLM MCP Zero Trust signer](https://docs.litellm.ai/docs/mcp_zero_trust)
- [LiteLLM `v1.100.1`](https://github.com/BerriAI/litellm/releases/tag/v1.100.1)
- [Okta developer guides](https://developer.okta.com/docs/guides/)
- [Okta API reference](https://developer.okta.com/docs/reference/)
- [OAuth for Okta API Services](https://developer.okta.com/docs/guides/implement-oauth-for-okta-serviceapp/main/)
- [Okta Applications API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/application)
- [Okta Authorization Servers API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/authorizationserver)
- [Okta System Log API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/systemlog)
- [Okta Help: Secure AI](https://help.okta.com/oie/en-us/content/topics/ai-agents/ai-agents-home.htm)
- [Okta blog: Securing AI Agents Beyond the Gateway](https://www.okta.com/blog/ai/securing-ai-agents-identity-architecture/)
- O4AA technical reference: token flows, agent identity/registry, connections and credentials, A2A, management API, product capabilities, and agentic-enterprise blueprint
- MCP Bridge technical reference: setup/access, admin API, 0.16 runtime authorization and tools, release/patch notes, and troubleshooting
- Local Postman collection matrix supplied for planning (read-only; collections and environments are not reproduced)
- Private comparative LiteLLM/O4AA demonstration repository supplied by a collaborator (architecture and operational lessons only; source, tenant data, credentials, and private link are not reproduced)

## Source snapshots inspected

- LiteLLM stable source export `v1.100.1` (`pyproject.toml` declares `1.100.1`)
- LiteLLM development snapshot declaring `1.102.0`
- GitHub release metadata distinguishing stable, release candidate, and dev builds

The development snapshot was used to identify likely future improvements and integration seams only. Customer-facing recommendations are based on stable behavior unless explicitly labeled otherwise.

The publishable MCP Bridge endpoint and workflow guide was checked against a local proprietary `0.16.6` source snapshot on 2026-09-11. No Bridge source, private repository link, internal schema, or implementation excerpt is included. Stable public detailed Bridge API documentation was not available; deployed-version behavior must be confirmed with the Bridge owner.

The comparative demonstration evaluated an older LiteLLM/Bridge combination and successfully proved a Bridge-protected MCP route, not native LiteLLM MCP enforcement. Its transferable lessons and version limitations are recorded in [REFERENCE-IMPLEMENTATION-LEARNINGS.md](REFERENCE-IMPLEMENTATION-LEARNINGS.md).

The Okta gateway article is treated as an architecture and customer-positioning source, not a product/API contract. Its last-mile identity and entitlement-ceiling implications are mapped into this repository in [IDENTITY-ARCHITECTURE-LEARNINGS.md](IDENTITY-ARCHITECTURE-LEARNINGS.md). Build claims still require public help/API evidence and tenant validation.

## Evidence classification

| Label | Meaning |
|---|---|
| Verified stable | Present in the selected stable release and source/docs |
| Prerelease | Present only in RC/dev/main at review time |
| Proposed integration | Design work that must be implemented and tested |
| O4AA product capability | Described by the current internal O4AA technical reference |
| Protocol specimen | Observed in Postman; may contain mistakes and is not production guidance |
| Architecture/positioning | Official problem framing or design guidance; not a versioned product/API contract |
