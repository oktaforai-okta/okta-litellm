# Build start here

This document is the handoff contract for a human or coding agent pointed at this repository. The repository contains the plan and acceptance criteria, not a pre-authorized deployment. Complete intake, select one topology, and then execute [the phased implementation plan](docs/IMPLEMENTATION-PLAN.md).

## Required user intake

Record answers without secrets. Store secret values only through the selected secret manager or an ignored local file.

| # | Ask the user | Required answer |
|---|---|---|
| 1 | Where should LiteLLM be built? | Local Docker, AWS, other cloud/Kubernetes, or existing service; account/project, region, network, DNS/TLS, PostgreSQL/Redis, secret manager, model provider, and budget |
| 2 | Which Okta environment should be used? | Non-production Okta base domain, O4AA availability, test users/groups, org/custom authorization servers, and environment owner |
| 3 | Which MCP Bridge should be used? | Gateway/Admin UI URLs, version, deployment owner, active PEP versus fallback/reference; `none` is valid for LiteLLM-first |
| 4 | How will Bridge API access be provided? | Human Admin UI OAuth token or allowlisted API Services app; secure delivery mechanism and mutation approval boundary |
| 5 | How will Okta administration/API access be provided? | Human role or service app, OAuth scopes, private-key/secret storage, and authorization to create or change O4AA/authorization-server objects |
| 6 | Where is authoritative Okta API information? | Public links in [OKTA API and access](docs/OKTA-API-AND-ACCESS.md) plus any tenant-specific O4AA help/API material the user is permitted to provide |
| 7 | Which Bridge API operations may be used? | Confirm Bridge version and authorize read-only discovery, import/sync, resource linking, or other exact mutations separately |
| 8 | What are the agent names and owners? | Separate exact names, purposes, owners, WLPs, client modes, and lifecycle expectations for Claude Code and the custom agent UI |
| 9 | Which use case/topology is required? | LiteLLM-first or hybrid; identify the MCP resource and the customer outcome to demonstrate |

After intake, present a short execution brief containing the selected topology, components, identities, trust boundaries, credentials by reference, phases to run, stop conditions, demo script, and cleanup owner. Obtain confirmation before making tenant, Bridge, or cloud mutations.

## Choose the topology

### A. LiteLLM-first

```text
Claude Code or Custom Agent UI
  -> LiteLLM model endpoint -> approved model provider
  -> LiteLLM MCP endpoint -> Okta ID-JAG/XAA -> protected MCP resource
```

Okta is the identity and token-issuance PDP. LiteLLM is the model router and primary MCP gateway PEP. The MCP resource remains the final PEP and validates the Okta token. MCP Bridge is not in the active MCP path; it remains the reference/fallback.

### B. Hybrid

```text
Claude Code or Custom Agent UI
  -> LiteLLM model endpoint -> approved model provider
  -> MCP Bridge -> Okta ID-JAG/XAA or STS -> protected MCP resource
```

Okta is the identity and token-issuance PDP. MCP Bridge is the MCP PEP. LiteLLM is the model router. This is the preferred fallback for third-party STS/consent, vaulted credentials, or when LiteLLM's native actor/binding gates fail.

## Required customer experiences

Build and validate both, without sharing logical agent credentials:

1. **Claude Code:** demonstrates a third-party MCP client, user login, the selected logical agent identity, model routing through LiteLLM, protected tool execution, evidence, and policy/lifecycle denial.
2. **Custom agent UI:** demonstrates the same governed controls in a first-party experience, including visible user/agent/resource context, explicit allow/deny state, and a link or reference to audit evidence. The UI must not display raw tokens or imply that model text is an Okta authorization decision.

See [DEMO-EXPERIENCES.md](docs/DEMO-EXPERIENCES.md) for acceptance criteria.

## Authoritative external starting points

- LiteLLM source: <https://github.com/BerriAI/litellm>
- LiteLLM documentation: <https://docs.litellm.ai/docs>
- LiteLLM MCP overview: <https://docs.litellm.ai/docs/mcp>
- LiteLLM Okta ID-JAG guide: <https://docs.litellm.ai/docs/mcp_id_jag>
- Okta developer guides: <https://developer.okta.com/docs/guides/>
- Okta API reference: <https://developer.okta.com/docs/reference/>
- Repository-specific source index: [RESEARCH-SOURCES.md](docs/RESEARCH-SOURCES.md)

Detailed public MCP Bridge implementation documentation is not generally available because the Bridge is proprietary. Use the publishable, source-verified operational contract in [MCP-BRIDGE-API.md](docs/MCP-BRIDGE-API.md), verify it against the deployed version, and involve the Bridge deployment owner when behavior differs.

