# MCP Bridge API and access guide

This is publishable operational documentation for integrating with an already deployed Okta MCP Bridge. It contains no MCP Bridge source code. The route and behavior inventory was checked against Bridge `0.16.6` on 2026-09-11; the deployed health version, image digest/build identifier, database schema, and local patches remain authoritative.

Detailed public Bridge API documentation is not currently available at a stable public developer URL. The Bridge is proprietary. If the running version differs or an endpoint is absent, stop and obtain the matching documentation from the Bridge deployment owner or Okta Professional Services rather than inferring a payload from this repository.

## Distinguish the two OAuth boundaries

| Boundary | Who uses it | Purpose | Credential |
|---|---|---|---|
| MCP client OAuth | Claude Code or the custom agent UI | Authenticate the user/client to the Bridge MCP endpoint | Interactive OAuth session issued through the Bridge/Okta flow |
| Bridge Admin API | Human administrator or automation service app | Inspect or configure agents, resources, connections, tools, and audit state | Okta org-authorization-server access token |

An MCP client token is not automatically an Admin API token. A LiteLLM key, model-provider key, Custom-AS resource token, SSWS token, and ID-JAG are also not substitutes for the Bridge Admin API token.

## Obtain Admin API access safely

Choose one method during build intake.

### Human administrator

The operator signs in to the Bridge Admin UI through its configured Okta SPA. The resulting short-lived Okta org-AS access token can authorize `/api/admin/*` when the Admin UI client ID is allowlisted by that Bridge. The operator should place the token in an approved secret manager or ignored local file/environment variable for the current session. Do not paste it into chat, print it, commit it, or retain it after expiry.

If a token is rejected, verify its issuer, expiry, and client ID against the Bridge's configured Admin UI allowlist. A successful browser login followed by Admin API `401` usually means the gateway does not trust that UI client ID, not that the user's primary login failed. The deployment owner must correct the Bridge allowlist.

### Okta API Services application

For unattended or repeatable automation, use an Okta API Services Integration with `client_credentials` and `private_key_jwt`. Its client ID must be allowlisted by the deployed Bridge. Keep the private JWK in the selected secret manager and rotate it with an overlap period.

Grant only the Okta management scopes required by the authorized operation. The normal 0.16 discovery/import surface may require:

- `okta.aiAgents.read`
- `okta.aiAgents.manage`
- `okta.apps.read`
- `okta.groups.read`
- `okta.authorizationServers.read`

Add `okta.apps.manage` only when explicitly authorizing linked-application credential operations. Identity scopes such as `openid`, `profile`, and `email` do not belong in a client-credentials request.

The token must come from the same Okta tenant's **org authorization server**, not a Custom Authorization Server. A token that mints successfully can still receive `401` if its `cid` is not allowlisted, or `403` when the caller lacks an Okta scope needed by a route.

## Required version discovery

Before any Admin API operation, record:

1. `GET /health` response version.
2. Deployed image reference and immutable digest/build identifier from the deployment owner.
3. Database schema level and any customer patch.
4. Okta tenant domain and the Bridge gateway/Admin UI base URLs.
5. Whether the environment implements the 0.16 master-resource model described below.

Never apply a 0.15 resource-shell repair procedure to 0.16. In 0.16, resources are admin-owned masters and each synced agent connection is explicitly linked to a master.

## Safe read-only discovery sequence

Run these reads first and retain only non-secret fields.

| Order | Method and path | What it establishes |
|---|---|---|
| 1 | `GET /health` | Running version and basic service health |
| 2 | `GET /api/admin/agents` | Bridge agent IDs, enabled state, and Okta workload-principal linkage |
| 3 | `GET /api/admin/resources` | Master MCP resources, exact URLs, protocols, and enabled state |
| 4 | `GET /api/admin/agents/{agent_id}/connections` | Okta connection IDs, active state, scopes/type, and master-resource linkage |
| 5 | `GET /api/admin/connections/status` | Last synchronization state and unresolved records |
| 6 | `GET /api/admin/agents/{agent_id}/tools` | Observed inventory for diagnosis only; not proof of authorization |
| 7 | `GET /api/admin/audit` | Available Bridge-side administrative evidence |

Tool visibility or count is never authorization proof. Prove a protected `tools/call`, resource-token admission, and downstream enforcement for each user/agent/connection combination.

## Publishable 0.16 endpoint map

This table intentionally documents operations without reproducing proprietary implementation or internal schemas.

| Area | Read operations | State-changing operations |
|---|---|---|
| Agents | `/api/admin/agents`, `/api/admin/agents/{agent_id}` | Create/update/delete agent; credential and proxy-secret operations |
| Okta import | `/api/admin/okta/agents`, agent detail, delegation-app candidates, potential connections | Import/bulk import agent, sync agent/all, create Okta-side connection |
| Resources | `/api/admin/resources`, resource detail, isolation/merge candidates | Create/update/delete/merge master resource |
| Agent connections | `/api/admin/agents/{agent_id}/connections` | Link or unlink a connection to/from a master resource |
| Synchronization | `/api/admin/connections/status` | Trigger connection or Okta synchronization |
| Tools | Global/resource/agent tool inventory and discovery state | Trigger discovery; classify, enable/disable, or annotate tools |
| Audit/events | Global, agent, resource, and export views | Export may expose sensitive operational data and requires handling approval |
| Sessions | Token-store statistics and visible-agent diagnosis | User force-logout or token-store deletion |
| Client registration/policy | DCR registration and policy reads | Revoke/unlink registration or change redirect trust policy |
| Optional authorization | Cedar policy/schema/entity/analytics reads when enabled | Create/update/delete/enable/disable policy; simulation and NL-authoring actions |

All POST, PUT, PATCH, and DELETE calls are mutations, including “sync,” “discover,” “import,” “simulate,” and “export” operations that may update caches, inventory, audit state, or external Okta objects. Obtain approval for the exact environment, route, target IDs, and intended result before calling them.

## 0.16 resource workflow

For an authorized new MCP target:

1. Confirm the Okta AI Agent exists, is active, has the correct owner/credential, and has an active Resource Connection.
2. Import or synchronize the agent only after that mutation is approved.
3. Re-read the agent's connections and capture the exact active `connection_id`.
4. Create or select one Bridge master resource with the exact MCP Streamable HTTP endpoint and enabled state.
5. Link the agent connection to that master through the connection-resource link operation.
6. Re-read the master and agent connection. Confirm the persisted link, URL, enabled state, connection type, audience/resource indicator, and scopes.
7. Authenticate a real test user and separately test `tools/list` and permitted/denied `tools/call` operations.

Synchronization does not prove a 0.16 connection is linked. An imported response or visible tool name does not replace the re-read and runtime test.

## Claude Code access

Confirm the imported Bridge agent is configured for the deployed release's supported Claude Code client mode. Current 0.16 guidance uses Claude's Client ID Metadata Document at `https://claude.ai/oauth/claude-code-client-metadata`. When several Bridge agents share that client metadata identity, the client must provide the deployment-approved deterministic agent selector; verify that selector cannot be caller-forged into another agent's privileges.

Claude Code registers the Bridge MCP URL and completes browser authentication through `/mcp`. For a path-scoped endpoint, the URL's resource slug must exactly match the configured master route. The user's Bridge session identifies inbound access; the selected workload principal and Resource Connection govern the outbound XAA/STS exchange.

## Custom agent UI access

Choose and document the custom UI's supported OAuth client-registration mode with the Bridge owner. Give it its own Okta application/client identity and its own O4AA agent/WLP. Do not reuse Claude's logical agent, Client ID Metadata Document, session, or signing credentials. The UI must use authorization code with PKCE for a browser-based public client and must never receive a downstream resource token or Bridge administrator token.

## Verification and failure triage

| Symptom | Verify first |
|---|---|
| Admin UI briefly loads, then returns to sign-in | Org-AS issuer, token expiry, Admin UI `cid`, and Bridge allowlist |
| Agent exists but no resource is usable | Agent enabled/active, Okta connection active, explicit master link, exact URL, master enabled |
| Connected but zero tools | Real MCP endpoint, connection/master link, authenticated cold discovery, then tool inventory |
| Tool visible but invocation denied | Per-call token admission, connection scopes/type, policy event, and downstream resource validation |
| STS resource shows no consent page | Selected agent, active linked STS connection, resource indicator, prior consent, and callback evidence |
| Import/sync returns `403` | Admin token's Okta API grants and tenant match |
| Admin API returns `401` | Org-AS issuer, expiry, signature/JWKS, and allowlisted UI/service-app client ID |

## Proprietary-source boundary

- Never copy Bridge source files, tests, internal comments, container contents, or proprietary implementation excerpts into this repository, issues, logs, or prompts.
- Do not publish private repository URLs, deployment credentials, customer tenant identifiers, or unredacted response bodies.
- API paths, HTTP methods, required scopes, version compatibility notes, original diagrams, and externally observable operational guidance are permitted under the project requirement.
- If more detailed request/response schemas are required and no approved public documentation exists, record that documentation gap and request release-specific documentation from the Bridge owner. Do not reverse-engineer and publish the missing schema.

