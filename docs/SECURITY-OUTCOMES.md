# Security outcomes and threat model

## What Okta adds

The value claim is precise: O4AA adds an Okta-governed workload principal with owner and lifecycle, a managed agent-to-resource relationship, Okta policy evaluation during token issuance, a short-lived user-plus-agent delegation artifact, and Okta System Log evidence. LiteLLM remains valuable for model routing, gateway admission, MCP aggregation, budgets, guardrails, and operational logging.

## Required security outcomes

1. **First-class logical agent identity:** the Okta actor is the dedicated customer-controlled `Claude-via-LiteLLM` workload principal, not an arbitrary header, shared LiteLLM key, team, or generic gateway. Runtime/binary attestation is a separate control.
2. **Dual custody:** the final token binds the authorized human subject and agent actor.
3. **Least privilege:** the token is audience-bound, short-lived, and scoped only for the requested demo capability.
4. **No confused deputy:** a caller cannot select another agent credential, route, resource, or subject.
5. **Defense in depth:** LiteLLM admission, Okta issuance, and Swiss Army validation each fail independently.
6. **Lifecycle control:** policy, connection, or agent changes prevent fresh authorization within a declared cache window.
7. **Auditable outcome:** action, decision, identity, scope, and result are correlated without recording credentials.

## Threats and mitigations

| Threat | Mitigation and proof |
|---|---|
| Forged user or agent identity | Validate Okta tokens; derive actor from bound route/credential; ignore caller agent headers |
| Shared gateway collapses agent lineage | Dedicated MVP route/WLP; later per-caller credential selector |
| Default-open MCP permissions | Explicit key/agent/server/tool grants; `require_key_mcp_access_defined`; negative raw calls |
| Tool hidden but still callable | Call-time authorization plus resource enforcement; invoke hidden name directly in tests |
| Gateway bypass | Private/allowlisted resource network plus mandatory JWT/audience/scope validation |
| Broad cached token outlives policy change | Least-privilege scope, short TTL, cache invalidation test, accurate kill-switch language |
| Token replay across resources | Audience/resource validation and short expiry; sender-constraining if the tenant/resource supports it |
| Static credential fallback | No fallback branch; token failure ends the request |
| `tools/list` leaks metadata | Single safe MVP server and explicit local allowlist; build list-time PDP hook for strict target |
| Sensitive logs | Structured allowlist schema; hash sensitive arguments; automated secret scans |
| Stale/missing evidence | Durable controlled sink; minimal synchronous receipt before sensitive work |
| Upstream/source drift | Pin tag, commit, image digest; rerun gates before upgrades |
| Master/encryption key loss | Managed rotation procedure, backups, and recovery test |
| Object-level overreach | Tool-handler checks or Okta FGA in addition to OAuth scopes |
| Copied LiteLLM credential impersonates user/agent route | Short-lived user/route-bound admission, non-sharing, revocation, cross-host replay test; add proof-of-possession/attestation for a runtime claim |
| Cross-user assertion or token-cache contamination | Cache keys include immutable tenant/user/agent/resource/scope/credential generation; concurrent-user and restart tests |
| Direct model or MCP bypass | Restricted Claude profile, private/allowlisted resource ingress, egress policy, explicit bypass tests |
| MCP tool poisoning or prompt injection | Treat metadata/results as untrusted; pin/approve servers and tool schemas; validate arguments; never let content select credentials, routes, or approvals |
| SSRF or DNS rebinding through MCP URL | Static destination allowlist, scheme/port restrictions, DNS/IP validation, private-address policy, controlled egress |
| Inbound header collision or raw bearer leakage | Strip/overwrite protected headers; separate admission and egress auth; forced-error/header fuzz tests |
| Admin/control-plane takeover | Separate deployment, identity, O4AA, database, evidence, and model-provider admin roles; MFA/JIT access and audited break-glass |
| Tool-loop, budget, or retry abuse | User/agent/route rate limits, tool-call budgets, idempotency, bounded retries, and loop detection |
| Model-provider data egress | Classify prompts/tool data, minimize transmission, approve provider/region/retention, and redact secrets before inference |
| Resource Connection/LiteLLM drift | Reviewed mapping ledger, automated comparison where possible, alerting, and fail-closed unknown mappings |

## Claim limitations

- Agent deactivation normally prevents **new token issuance**. A previously issued or cached bearer may remain valid until expiration unless the system provides active revocation and checks it.
- An Okta token grant proves policy at issuance time, not that every tool parameter was safe. Validate parameters and business objects separately.
- LiteLLM's self-signed MCP JWT mode is not the O4AA chain. Its `act` value may represent a team or organization and is not an Okta-issued agent actor.
- LiteLLM internal `agent_id` is not automatically an O4AA workload principal.
- A one-agent route is an MVP isolation technique, not a fleet architecture.
- The WLP attests possession of its private key by the LiteLLM route boundary; it does not prove the vendor Claude executable made a request.
- A retained OIDC assertion establishes whose delegated session/permissions were used. It does not prove fresh human presence or per-tool consent unless step-up/HITL is implemented.
- If an authorized bearer can reach Swiss Army around LiteLLM, LiteLLM is one gateway PEP rather than a mandatory enforcement point.

## Evidence schema

Minimum sanitized event fields:

```text
timestamp, correlation_id, litellm_request_id, model_request_id,
tenant_id, human_subject, agent_principal,
resource_id, resource_audience, scopes,
mcp_server, tool_name, arguments_hash,
decision_stage, decision_outcome, okta_event_id,
token_jti_hash, upstream_status, side_effect_id,
policy/config revision, cache_status,
session_capture_time, credential_owner_id, task_reason_id
```

Never emit token bodies, authorization headers, ID-JAG assertions, refresh tokens, client assertions, private JWKs, model-provider keys, raw Postman variables, or sensitive tool inputs/outputs.

## Data and credential lifecycle

Inventory stored identity assertions, any refresh material, virtual/admission keys, WLP keys, model-provider credentials, token/decision caches, configuration, prompts/tool data, and evidence. For each, record classification, owner, location/region, encryption and key separation, TTL, retention/deletion, backup inclusion, restore procedure, access audit, rotation, offboarding purge, and incident response. Do not request `offline_access` unless renewal is implemented and this lifecycle is approved.

## Failure policy

- Okta denial or error: fail closed for the MCP action.
- Missing human assertion: require reauthentication.
- Upstream token rejection: one safe refresh/retry for 401; none for 403.
- Evidence failure: fail a sensitive demo action if no minimal receipt can be retained.
- Hybrid fallback: explicit and manual; a denied request never routes around Okta.
