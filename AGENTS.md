# Repository instructions for build agents

This repository is a gated implementation plan. Read [BUILD-START-HERE.md](BUILD-START-HERE.md), [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), [docs/IDENTITY-ARCHITECTURE-LEARNINGS.md](docs/IDENTITY-ARCHITECTURE-LEARNINGS.md), [docs/IMPLEMENTATION-PLAN.md](docs/IMPLEMENTATION-PLAN.md), and [docs/REFERENCE-IMPLEMENTATION-LEARNINGS.md](docs/REFERENCE-IMPLEMENTATION-LEARNINGS.md) before creating runtime code or changing any environment.

## Mandatory intake

Do not begin the build until the user has answered all nine items below. Ask them together, record the answers in an untracked/local build-intake record, and restate the selected topology and environment boundaries before making changes.

1. Where should LiteLLM run: local Docker, AWS, another cloud, Kubernetes, or an existing LiteLLM deployment? Obtain the account/project, region, network, DNS/TLS, secret-store, database, and model-provider constraints that apply.
2. Which non-production Okta environment should be used? Obtain its base domain, O4AA entitlement/status, allowed test users/groups, and authorization-server constraints.
3. Which MCP Bridge should be used? Obtain the gateway URL, Admin UI URL, reported version, deployment owner, and whether it is only the reference/fallback or the active MCP PEP. `None` is valid for the LiteLLM-first topology.
4. How will the builder authenticate to the MCP Bridge Admin API? Choose a short-lived human Admin UI OAuth access token or an allowlisted Okta API Services app using `client_credentials` and `private_key_jwt`. Never request a token in chat or commit one; use an approved secret manager or local ignored file.
5. How will the builder access and configure the Okta environment? Identify the human admin role or OAuth service app, approved authentication method, granted management scopes, secret-delivery path, and whether tenant mutations require a separate approval.
6. Which documentation is authoritative for this tenant and release? Start with [docs/OKTA-API-AND-ACCESS.md](docs/OKTA-API-AND-ACCESS.md), record any tenant-specific/private O4AA documentation, and do not invent schemas where public detailed documentation is unavailable.
7. Which MCP Bridge API version and operations are authorized? Follow [docs/MCP-BRIDGE-API.md](docs/MCP-BRIDGE-API.md), begin with its read-only discovery sequence, and obtain explicit approval before POST, PUT, PATCH, or DELETE operations.
8. What are the exact agent names, types, owners, and client experiences? At minimum decide names for the Claude Code logical deployment identity and the custom agent UI identity; never silently share one WLP/credential between distinct agents.
9. Which use case is being built?
   - LiteLLM-first: Okta is the identity/token PDP, LiteLLM is the model router and primary MCP gateway PEP, and the MCP resource is the final PEP.
   - Hybrid: Okta is the identity/token PDP, MCP Bridge is the MCP PEP, and LiteLLM remains the model router.

If an answer is unknown, pause the affected phase at its documented gate. Do not replace an unknown with a production assumption.

## Non-negotiable boundaries

- Never publish, copy, vendor, quote, or reconstruct MCP Bridge source code. The Bridge is not open source.
- Publishable material is limited to approved API documentation, endpoint names, configuration guidance, externally observable behavior, architecture, and original integration code written for this repository.
- Never commit Okta or Bridge tokens, private JWKs, client secrets, Postman environments, packet captures, tenant exports, customer identifiers, or proprietary container/image contents.
- Use the pinned LiteLLM release and repeat the capability/license gates before changing versions.
- Treat `Claude-via-LiteLLM` as a customer-controlled logical route identity unless runtime attestation is separately implemented and proven.
- Never route around an Okta or gateway denial. Hybrid fallback is a manual topology change with fresh authentication.
- Make Okta's value visible in every demonstration: governed workload principal, owner/lifecycle, Resource Connection, human-plus-agent token lineage, policy-driven issuance, resource enforcement, System Log evidence, and deactivation behavior.

## Build behavior

- Implement one phase at a time and attach its exit artifact before continuing.
- Start all environment work with read-only discovery and an exact version/image/config inventory.
- Use synthetic users and data in isolated non-production environments.
- Keep Claude Code and custom agent UI model/MCP credentials, identities, telemetry, and test evidence separate.
- Do not claim MCP Bridge parity, per-tool Okta decisions, instantaneous bearer revocation, or Claude binary attestation unless the corresponding validation gate passes.
