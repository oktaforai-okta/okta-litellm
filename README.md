# Okta + LiteLLM MCP enforcement plan

Planning and build-handoff repository for customer demonstrations in which Okta for AI Agents (O4AA) is the governed identity and policy/token decision plane while LiteLLM remains the customer's model router.

This repository currently contains a plan, not a working implementation. Build agents must complete [the mandatory nine-question intake](BUILD-START-HERE.md) before creating code or changing LiteLLM, Okta, MCP Bridge, cloud, or MCP-resource configuration.

## Recommendation

Proceed with a gated LiteLLM-first demo using Claude Code, a custom agent UI, and the Swiss Army MCP. Pin LiteLLM to stable `v1.100.1` and one verified container digest. Prove **Claude-via-LiteLLM** first, then add the custom UI as a separate customer-controlled O4AA workload principal with its own dedicated LiteLLM route and credential. These bindings attribute calls to configured logical routes; they do not attest a client executable unless separate workload proof-of-possession is added.

The demo is successful only if both Claude traffic planes traverse LiteLLM, the final resource token proves the human subject and intended logical workload actor, Okta policy controls issuance, the Swiss Army MCP rejects invalid or insufficient tokens, and one evidence bundle reconstructs the action across all three systems without ambiguous time-window joins.

Do not claim complete MCP Bridge replacement. The first milestone proves one human-context XAA flow to an internal resource. Okta STS brokered consent, OPA vaulted secrets, Resource Connection synchronization, and safe multi-agent credential selection remain gaps. The supported fallback is LiteLLM for model routing plus MCP Bridge for MCP identity and credential brokering.

## Supported use cases

1. **LiteLLM-first:** Okta is the identity/token PDP, LiteLLM is the model router and primary MCP gateway PEP, and the protected MCP server is the final resource PEP.
2. **Hybrid:** Okta is the identity/token PDP, MCP Bridge is the MCP PEP, and LiteLLM remains the model router.

Both use cases must demonstrate Claude Code and a separately registered custom agent UI. The customer story centers Okta's workload-principal identity, owner/lifecycle, Resource Connection, scoped human-plus-agent token lineage, System Log evidence, and predictable deactivation of fresh authorization.

## Start a build

- Read [BUILD-START-HERE.md](BUILD-START-HERE.md).
- Follow the repository-agent rules in [AGENTS.md](AGENTS.md); Claude Code also receives [CLAUDE.md](CLAUDE.md).
- Select the topology, environment, Bridge, access methods, agent names/owners, and customer use case with the user.
- Execute [the implementation plan](docs/IMPLEMENTATION-PLAN.md) one gate at a time.

LiteLLM source is at [BerriAI/litellm](https://github.com/BerriAI/litellm); product documentation is at [docs.litellm.ai](https://docs.litellm.ai/docs).

## Read this first

- [Architecture](docs/ARCHITECTURE.md)
- [Build intake and handoff](BUILD-START-HERE.md)
- [Demo story](docs/DEMO-STORY.md)
- [Claude Code and custom UI experiences](docs/DEMO-EXPERIENCES.md)
- [Implementation plan](docs/IMPLEMENTATION-PLAN.md)
- [Validation plan](docs/VALIDATION-PLAN.md)
- [LiteLLM feasibility](docs/LITELLM-FEASIBILITY.md)
- [Postman correlation](docs/POSTMAN-CORRELATION.md)
- [Security outcomes](docs/SECURITY-OUTCOMES.md)
- [NFR and operations plan](docs/NFR-AND-OPERATIONS.md)
- [Okta API and access guide](docs/OKTA-API-AND-ACCESS.md)
- [MCP Bridge API and access guide](docs/MCP-BRIDGE-API.md)
- [Gap audit](docs/GAP-AUDIT.md)
- [Comparative implementation learnings](docs/REFERENCE-IMPLEMENTATION-LEARNINGS.md)
- [Open questions](docs/OPEN-QUESTIONS.md)
- [Decision log](docs/adr/0001-litellm-first-pep.md)

## Scope boundary

In scope: planning, architecture, acceptance gates, threat and failure analysis, source/release provenance, and a hybrid rollback path.

Out of scope for this planning pass: runtime code, tenant provisioning, infrastructure deployment, secrets, copied Postman environments, and customer claims of production parity.

MCP Bridge is proprietary and is not open source. This repository may contain original API/setup documentation and externally observable behavior, but it must never contain MCP Bridge source, copied implementation excerpts, private repository content, container contents, or customer deployment secrets. Detailed public Bridge API schemas are not currently available at a stable public URL; obtain release-matched documentation from the Bridge owner when this guide is insufficient.

## Decision summary

| Question | Decision |
|---|---|
| Primary resource | Swiss Army MCP under an Okta Custom Authorization Server |
| Primary client | Claude Code |
| Claude model path | `ANTHROPIC_BASE_URL` + user-bound LiteLLM credential; independently observable from MCP |
| LiteLLM baseline | Stable `v1.100.1`, later stable only after repeating all gates |
| Human flow | Okta Org-AS ID token, preferably captured through LiteLLM generic OIDC SSO |
| Agent flow | Separate O4AA workload principals, credentials, and routes for Claude Code and the custom UI |
| Enforcement | LiteLLM admission/routing/allowlists + Okta token decision + resource-side JWT/scope enforcement |
| Secondary scenario | GitHub only after the Okta STS gap is resolved or through MCP Bridge |
| Fallback/hybrid | LiteLLM model routing + MCP Bridge MCP enforcement |
| Required clients | Claude Code plus a separately governed custom agent UI |

## Repository hygiene before implementation

Choose a license and confirm named owners before accepting code. Enable branch protection and secret scanning, then add implementation CI, dependency/image pinning, and required security checks. The repository already forbids raw tokens, private keys, Postman environments, packet captures, and proprietary MCP Bridge source.
