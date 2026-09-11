# Okta API and environment access

This guide tells a build agent what access to request and where to verify Okta API behavior. It does not grant authority to modify a tenant.

## Intake for the Okta environment

Ask the user for:

1. The exact non-production Okta base domain and environment owner.
2. Confirmation that Okta for AI Agents/O4AA is enabled and which features are GA, EA, Preview, or unavailable in that tenant.
3. Test users, groups, and administrators; never use customer production identities for the demo.
4. The approved Custom Authorization Server, audience, resource indicator, scopes, access policies, and whether new objects may be created.
5. Whether access is through a human administrator or an Okta API Services application.
6. The granted OAuth management scopes and the approval boundary for POST, PUT, PATCH, DELETE, activation, deactivation, and key operations.
7. How secrets/private keys will be delivered by reference through an approved secret manager or ignored local file.
8. Where tenant-specific O4AA API/help documentation can be accessed if the public reference is incomplete.

## Preferred access models

### Human administration

Use a named administrator with MFA and the least role that can complete the approved work. Current O4AA management operations may require elevated or Super Administrator privileges depending on feature/release; verify in the target tenant. Use the Admin Console for interactive object creation and lifecycle steps when repeatable automation is not required.

### API Services application

For automation, use an Okta API Services Integration with `client_credentials` and `private_key_jwt`. Store the private key outside the repository, assign least-privilege Okta API scopes, rotate keys with overlap, and attribute activity to the service app. Tokens must come from the org authorization server.

Common scopes for this plan are:

| Scope | Purpose |
|---|---|
| `okta.aiAgents.read` | Read AI Agents, credentials metadata, and connections |
| `okta.aiAgents.manage` | Register/update/lifecycle agents, connections, and public keys |
| `okta.apps.read` | Read linked OIDC application metadata |
| `okta.apps.manage` | Only when approved app/credential changes are required |
| `okta.groups.read` | Verify eligibility and assignments |
| `okta.authorizationServers.read` | Inspect Custom Authorization Servers, scopes, policies, and rules |
| `okta.authorizationServers.manage` | Only when authorized to create/change those objects |
| `okta.logs.read` | Retrieve Okta System Log evidence |

The actual endpoint can require additional scope or role constraints. Treat `403` as an authorization boundary to resolve with the tenant owner, not permission to broaden the service app automatically.

SSWS API tokens are not the default for this project. Use one only if the tenant owner explicitly approves it and OAuth service-app access cannot meet the task. Never paste any access token, client assertion, private JWK, or session cookie into chat or Git.

## O4AA management API contract

The tenant's workload-principal API uses the base path `/workload-principals/api/v1`. Relevant resource families include:

| Resource family | Purpose |
|---|---|
| `/ai-agents` | List, register, read, update, delete, activate, and deactivate agent workload principals |
| `/ai-agents/{agentId}/connections` | Manage Resource Connections and lifecycle state |
| `/ai-agents/{agentId}/credentials/jwks` | Manage agent public signing/encryption keys and key lifecycle |
| `/potential-connections` | Discover candidate resources for an agent |
| `/operations` | Poll asynchronous registration/lifecycle operations |

Read operations generally require `okta.aiAgents.read`; mutations require `okta.aiAgents.manage`. Agent mutation operations can be asynchronous, so an accepted request is not success until its operation reaches a completed state. Paginated list operations use the tenant response's opaque continuation link/cursor; do not construct cursors.

Before relying on the API, verify the exact tenant schema for agent status, connection type, scope condition, resource indicator, key status, async errors, and claim profile. Never copy example tenant IDs or private key material into configuration.

## Public documentation

Use these official entry points before third-party examples:

- [Okta developer guides](https://developer.okta.com/docs/guides/)
- [Okta API reference](https://developer.okta.com/docs/reference/)
- [OAuth and OpenID Connect concepts](https://developer.okta.com/docs/concepts/oauth-openid/)
- [OAuth for Okta API Services](https://developer.okta.com/docs/guides/implement-oauth-for-okta-serviceapp/main/)
- [Applications API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/application)
- [Authorization Servers API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/authorizationserver)
- [Groups API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/group)
- [System Log API](https://developer.okta.com/docs/api/openapi/okta-management/management/tags/systemlog)
- [Okta Help: Secure AI](https://help.okta.com/oie/en-us/content/topics/ai-agents/ai-agents-home.htm)

As of the repository review date, detailed public OpenAPI tag pages for O4AA AgentRegistration, AgentConnections, and AgentPublicKey were not available at stable developer.okta.com URLs. Builders must use the documentation exposed to the O4AA-enabled customer/partner tenant or obtain the current approved reference from the Okta account/product team. This repository intentionally provides the endpoint-family contract but does not republish private schemas.

## Read-first tenant discovery

Before mutation, produce a sanitized inventory of:

- tenant issuer/domain and feature availability;
- existing AI Agents, owners, status, linked applications, and public-key metadata;
- Resource Connections, type, active state, authorization server/resource, resource indicator, scope condition, and scopes;
- target Custom Authorization Server issuer, audience, claims, scopes, access policy, rules, assigned clients/groups, and token lifetime;
- test-user/group/app assignments;
- relevant System Log event visibility;
- existing object names/IDs that could collide with the planned demo.

Do not include raw tokens, private keys, secrets, user PII, or full tenant exports in the committed inventory.

## Mutation and verification rule

Every mutation needs the exact tenant, target object, intended state, rollback/cleanup action, and user authorization. Re-read the object after mutation and retain only sanitized evidence. Activation, deactivation, key changes, group/app assignments, connection creation/removal, and authorization-server policy changes are material mutations even when described as demo setup.

