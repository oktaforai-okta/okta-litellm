# Implementation plan

This is the execution backlog for a later build. Each phase ends at a gate; do not continue by interpreting a failed gate as success.

## Execution map

Effort ranges are planning estimates for a prepared non-production tenant, not commitments. Phase 10 is independent and optional.

| Phase | Depends on | Primary accountable role | Exit artifact | Estimate |
|---|---|---|---|---|
| -1 User intake/topology | Repository handoff | Solution architect | Confirmed nine-item execution brief | 0.5–1 day |
| 0 Facts/prerequisites | Intake approval | Solution architect | Capability ledger, trust diagram, RACI, approved NFRs | 2–4 days |
| 1 No-Okta control | 0A | Demo owner | Sanitized baseline evidence | 1 day |
| 2 Identity/policy plane | 0A + applicable 0B/0C | Okta/O4AA owner | Active object manifest + wire tests | 2–5 days |
| 3 LiteLLM baseline | 0A, 2 | LiteLLM/platform owner | Pinned deployment + mapping ledger | 2–4 days |
| 3H Hybrid MCP plane | 0C, 2, approved Bridge access | Bridge/identity owners | Versioned Bridge linkage + dual-client trace | 2–4 days |
| 4 Claude/user binding | 3 | Identity/LiteLLM owners | Dual-plane trace + binding report | 2–4 days |
| 5 Enforcement | 4 | Security/test owner | Positive/negative test report | 2–4 days |
| 6 Custody | Evidence contract, 5 | Observability owner | Reconstructable evidence bundle | 2–4 days |
| 7 Lifecycle | 5, 6 | Security/test owner | Measured stale-window report | 1–3 days plus TTL wait |
| 8 Operations/rehearsal | 3–7 | Service/demo owner | Load/recovery/runbook report | 2–4 days |
| 9 Direction decision | 0–8 | Product + architecture owners | Signed go/hybrid/stop ADR | 1 day |
| 10 Optional GitHub | Separate STS approval | Integration owner | STS-specific plan/evidence | Not estimated here |
| 11 Teardown | Demo complete | Service owner | Deletion/retention record | 1 day |

## Phase -1 — Complete user intake and select the topology

Before code or environment changes, ask and record all nine items in [BUILD-START-HERE.md](../BUILD-START-HERE.md): LiteLLM deployment location; Okta environment; MCP Bridge selection; Bridge OAuth access; Okta administrative/API access; authoritative Okta documentation; authorized Bridge API operations; exact agent names/owners; and selected customer use case.

Produce an execution brief that names the topology, Claude and custom-UI identities, target MCP resource, deployment/network boundaries, credential references, authorized mutations, demo outcome, stop conditions, and cleanup owner. Do not place secret values in the brief.

Gate -1: the user confirms the execution brief. Unknown environment or authority items remain blockers rather than implementation assumptions.

## Phase 0 — Freeze facts and prerequisites

Deliverables:

- Record LiteLLM `v1.100.1`, source commit, signed container image, and immutable digest.
- Confirm which required capabilities are open source versus licensed: MCP gateway, generic OIDC SSO, stored identity assertion, custom auth/hooks, and ID-JAG.
- Inventory the Okta preview tenant features and exact O4AA object/API availability.
- Select a synthetic Swiss Army environment and a non-production model provider.
- Choose repository license, owners, branch controls, CI, secret scanning, and disclosure policy.
- Choose one reproducible reference deployment and freeze its trust-boundary/data-flow diagram.
- Approve the evidence contract, data classification/retention schedule, mapping ledger, demo NFR targets, and named RACI in [NFR-AND-OPERATIONS.md](NFR-AND-OPERATIONS.md).
- Require SBOM generation, dependency/container vulnerability scanning, license review, and provenance verification; define an approved fallback when an upstream image is not signed.
- Write a capability ledger labelled `stable`, `prerelease`, `proposed integration`, or `O4AA product capability`.
- Review [the comparative implementation learnings](REFERENCE-IMPLEMENTATION-LEARNINGS.md) and carry each applicable negative test into the execution brief without copying its private implementation.

Gate 0A: the chosen LiteLLM artifact is reproducible and has all MVP features.

Gate 0B: a wire-level spike proves Okta accepts LiteLLM's private-key client assertion for the intended O4AA workload principal. The final token must show the intended actor—not a generic gateway actor.

If 0B fails, stop the native path. Design the credential-selector/`principal_id` extension or use MCP Bridge; do not rewrite the claim.

Gate 0C for the hybrid lane: the selected Bridge release/build is identified, its Admin API authentication succeeds with the approved human or service-app OAuth method, and the read-only discovery sequence confirms the expected 0.16 resource/connection model. If it fails, do not guess an older API workflow; obtain release-matched Bridge documentation or stop the hybrid lane.

## Topology branch

- **LiteLLM-first:** execute Phases 1–8 as written. MCP Bridge remains a read-only comparison/fallback unless the user separately authorizes the hybrid lane.
- **Hybrid:** LiteLLM handles the model plane; execute Phase 3H for the MCP plane, then apply the common enforcement, custody, lifecycle, operations, and demo gates to MCP Bridge instead of claiming native LiteLLM MCP enforcement.
- Never run both MCP gateways as an automatic per-request fallback. Changing topology is manual and requires fresh authentication and a new evidence baseline.

## Phase 1 — Establish the no-Okta control

Build a disposable LiteLLM-to-Swiss-Army path with only synthetic, read-only tools. Use a least-privilege virtual key and approved model aliases, not the LiteLLM master key or an unrestricted wildcard model route. Capture what LiteLLM already knows: user/key/team/agent, route, tool, model, time, and result. Explicitly document which O4AA artifacts are absent.

Gate: reviewers agree the comparison is factual and does not pretend LiteLLM lacks its own identity or audit features.

## Phase 2 — Provision the identity and policy plane

Create or verify:

1. Two test users and eligibility groups.
2. An Org-AS OIDC application for the human login subject.
3. Separate O4AA AI Agent/workload principals for `Claude-via-LiteLLM` and the user-approved custom UI name, each explicitly defined as a composite logical route identity with its own owner and lifecycle metadata.
4. Link the appropriate OIDC app on each agent's User Sign-on configuration, assign the demo group, activate each agent and signing key, and verify every expected status transition.
5. One active signing JWK per agent; retain each private key only in the deployment secret store/KMS boundary and prohibit reuse.
6. A Swiss Army Custom Authorization Server with explicit audience, resource indicator, read scope, claims, and short access-token lifetime.
7. A policy supporting the ID-JAG/JWT bearer grants and assigning the intended user/group, linked OIDC app, and AI Agent as required by the tenant.
8. One active O4AA Resource Connection per agent to the Custom AS/resource, with independently verified mapping and lifecycle.
9. System Log access for evidence collection.

Gate: reproduce the human private-key XAA wire flow outside LiteLLM with sanitized diagnostics, using collection 09 only as a protocol oracle. Freeze the tenant-derived `audience` and `id_jag_resource` values, their exact LiteLLM field mapping, and the tenant-issued human/agent claim profile. Validate `iss`, `aud`, human subject, logical agent actor, scope, `jti`, lifetime, and denials for an unassigned user, unlinked app, inactive WLP/key/connection, wrong audience, and wrong resource indicator. Never fabricate or customize an `act` claim merely to satisfy the demo narrative.

## Phase 3 — Deploy the pinned LiteLLM baseline

Use a dedicated route/configuration for each demo agent; begin with Claude and add the custom UI only after Claude passes. Required supporting services:

- PostgreSQL for durable LiteLLM state and stored SSO assertion;
- Redis only if the selected deployment mode requires it, with explicit cache TTL/invalidation behavior;
- TLS ingress and private network egress;
- secret manager references for LiteLLM master key, OIDC secret, model credential, and each agent's distinct WLP private key;
- structured logs to a controlled sink;
- readiness checks that do not disclose credentials.

Baseline configuration intent:

- generic Okta OIDC SSO with `openid profile email`; request `offline_access` only if a verified renewal design requires it and its storage/rotation risk is accepted;
- separate LiteLLM admission credentials mapped to the intended users and logical agents;
- one exact Swiss Army MCP server route/configuration per logical agent, each with its own WLP credential;
- `auth_type: oauth2_id_jag`;
- Org-AS token endpoint for leg one;
- Custom-AS token endpoint for leg two;
- exact, tenant-verified LiteLLM `audience` and `id_jag_resource` values;
- private-key JWT, not client-secret fallback;
- explicit server/tool allowlists and `require_key_mcp_access_defined`;
- no `delegate_auth_to_upstream` for the protected route;
- no static-credential fallback;
- only the least-privilege scope needed by the first tool.

Gate: configuration inspection shows no default-open path, no caller-controlled agent mapping, no token in logs, and no direct resource bypass. The mapping ledger matches the active WLP, connection, audience, scopes, route, key ID, validator profile, and config revision.

## Phase 3H — Configure the hybrid MCP plane

This phase replaces LiteLLM MCP configuration in Phases 3–5; LiteLLM model routing remains required.

1. Confirm the selected Bridge gateway/Admin UI URLs, deployment owner, health version, image/build identity, schema, and patch status.
2. Obtain the approved short-lived human or API Services OAuth access method described in [MCP-BRIDGE-API.md](MCP-BRIDGE-API.md). Never store the token in Git or chat.
3. Run the documented read-only discovery sequence before any import, sync, resource, connection, tool, credential, DCR, or policy mutation.
4. Register separate Okta workload principals and owners for Claude Code and the custom agent UI, with separate client bindings/credentials and active Resource Connections.
5. For Bridge 0.16, import/synchronize only when approved, create/select the master MCP resource, explicitly link each agent connection to the intended master, and re-read the persisted state.
6. Configure Claude's supported client mode and the custom UI's separate OAuth client mode for that exact Bridge version. Do not share Claude's CIMD/client identity with the custom UI.
7. Authenticate both clients and prove protected calls through XAA. Use STS only for a separately approved third-party provider scenario, including consent and cache/isolation tests.
8. Prove tool execution authorization separately from tool catalog visibility, then capture Bridge, Okta, and resource evidence.

Gate 3H: model traffic demonstrably uses LiteLLM; MCP traffic demonstrably uses the selected Bridge; each client resolves to its intended user, logical agent, connection, and resource; no proprietary Bridge source has entered the repository; and all mutations were approved and re-read.

## Phase 4 — Bind Claude and the human assertion

1. Sign the demo user into LiteLLM through **generic OIDC**, not a provider-specific or SAML login path.
2. Map immutable OIDC `iss` + `sub` to one LiteLLM user; never join identity by email alone.
3. Confirm the stored identity assertion is encrypted, mapped to the same user as a short-lived/revocable Claude credential, and survives a process restart without crossing users.
4. Configure Claude Code's model plane with the LiteLLM Anthropic-compatible endpoint and its intended model credential.
5. Configure Claude Code's MCP plane with the single-server LiteLLM MCP endpoint and the user/route-bound admission credential.
6. Keep LiteLLM admission material separate from egress `Authorization`. Prohibit direct-token mode in the customer demo unless the pinned build proves the inbound Org-AS token subject matches the bound LiteLLM user.
7. Validate `tools/list` and `tools/call` resolve the same subject under two simultaneous users.
8. Record issuance, TTL, client storage, non-sharing, rotation, revocation, logout, suspension, restart, and reauthentication behavior for the Claude credential and stored assertion.
9. Register the custom UI as a separate browser client using authorization code with PKCE and a backend-for-frontend where server-held credentials/exchanges are needed.
10. Bind the custom UI to its own LiteLLM user/client, dedicated MCP route, WLP signing credential, and evidence identity; repeat the list/call, restart, revocation, bypass, and cross-user tests.
11. Run Claude and the custom UI concurrently and prove there is no assertion, admission key, model/MCP credential, token cache, response, or evidence crossover.
12. Use separate client/project profiles for the intentionally unprotected control and governed path; never expose both similarly named MCP tools to the model in one profile.

Gate: Claude and the custom UI each list only the intended synthetic tools and invoke them through their expected human-plus-logical-agent chain. Independent telemetry proves both clients' model inference and MCP traffic traverse LiteLLM, while direct-provider/direct-MCP configurations and cross-client identity reuse fail.

Stable `v1.100.1` requires user re-login when its retained ID token expires. Assertion auto-renewal exists in later prerelease/main work but must not be represented as stable until it ships in a verified stable version.

## Phase 5 — Enforce at every boundary

Run the positive and negative matrix in [VALIDATION-PLAN.md](VALIDATION-PLAN.md). Confirm:

- LiteLLM admission fails before egress for an unbound key;
- Okta denial produces no final resource credential;
- LiteLLM never falls back to shared/passthrough credentials;
- Swiss Army rejects replay to a wrong audience or insufficient scope;
- direct resource calls are independently protected;
- object-level authorization occurs inside the tool handler for any mutable tool;
- a valid resource bearer cannot reach Swiss Army from an untrusted source, or the customer claim is downgraded from mandatory LiteLLM enforcement;
- a copied User A credential, raw JSON-RPC client, and User A key plus User B ID token cannot inherit misleading User A/Claude attribution;
- read and simulated-write tools on the same server prove which decisions belong to Okta scopes, LiteLLM's local ceiling, and the resource;
- inbound `Authorization`, identity, forwarding, host, and correlation header collisions cannot override trusted values.
- identity-like fields supplied in MCP tool arguments have no effect on subject, actor, scope, approval, or correlation;
- the complete scope set is tested; requesting an allowed and disallowed scope together never produces an undocumented partial grant;
- missing/wrong WLP, active `kid`, issuer, audience, resource indicator, or JWKS origin fails before a protected side effect;
- a stale downstream MCP session is invalidated and retried at most once without changing user, agent, credential, resource, or topology;
- invalid or unverified JWT data cannot populate a successful identity/custody record, and resource denials cannot be encoded as successful tool outcomes.

Gate: all required controls pass, including raw JSON-RPC attempts that bypass the Claude UI.

## Phase 6 — Produce chain-of-custody evidence

Define a common evidence schema before deployment and export sanitized LiteLLM action events. Generate one trusted high-entropy correlation ID at the gateway, propagate it to Swiss Army, and join token issuance through a hashed final-token `jti` and an Okta transaction/event identifier where exposed. A bounded time/subject/actor/resource tuple is contextual evidence only and may not satisfy the custody gate.

Gate: under concurrent, duplicated, delayed, and reordered calls, an independent reviewer can unambiguously answer whose delegated session and permissions were used, which logical agent route acted, against which resource/tool, under what scope/policy/config revision, for which sanitized task/reason identifier, and with what outcome—without seeing a secret or raw token. Do not imply per-action user presence unless step-up/HITL is actually implemented.

## Phase 7 — Prove policy change and deactivation

1. Warm every relevant token and decision cache; record the configured and observed TTL, including LiteLLM's behavior when `expires_in` is absent or malformed.
2. Deactivate the connection or agent, or remove user eligibility.
3. Without intervention or assumed eviction API, continue gateway calls and direct replay of the already-issued bearer until the final successful action; measure the stale-authorization window.
4. Force a demonstrable fresh issuance using a supported method—expiry or restart unless an eviction API is proven—and show the Okta denial and absence of a new resource side effect.
5. Repeat for user suspension, credential/session revocation, group removal, connection disablement, agent/key deactivation, and signing-key rotation.

Gate: the observed stale window is within the approved SLO and the fresh-exchange denial is visible. Describe the result as "blocks new authorized work after the measured cache/bearer window." Do not claim universal instantaneous bearer revocation.

## Phase 8 — Operationalize and rehearse

1. Instrument token-leg latency/errors, assertion expiry, cache hits/age, admission failures, 401/403s, PostgreSQL/Redis health, JWKS refresh, resource health, and evidence lag.
2. Build the dashboards, alerts, and owner-linked runbooks defined in [NFR-AND-OPERATIONS.md](NFR-AND-OPERATIONS.md).
3. Run concurrency, latency, outage, backup/restore, key rotation, privacy/secret leakage, and configuration-drift tests.
4. Rehearse the manual hybrid switch: pre-provision MCP Bridge, change Claude's MCP endpoint, clear incompatible sessions/caches, authenticate freshly, verify identity/evidence, and measure recovery time.
5. Produce support handoff, incident ownership, known limitations, and a bill-of-materials/cost record.

Gate: NFRs and recovery targets pass, every alert/runbook has an owner, and hybrid recovery completes within its approved RTO without routing around a denial.

## Phase 9 — Decide the production direction

Score three outcomes:

| Result | Next step |
|---|---|
| Native MVP passes and dedicated-agent scale is acceptable | Package the narrow demo and document the boundary |
| MVP passes but multi-agent/per-tool policy is required | Build a thin upstream-compatible principal/scope selector and list hook |
| Identity, STS, lifecycle, or assurance gates fail | Retain MCP Bridge for the MCP plane; LiteLLM remains the model router |

## Phase 10 — Optional GitHub scenario

Treat GitHub as a separate STS/brokered-consent workstream:

- register/configure the supported GitHub resource server connector;
- prove user consent and `interaction_required` handling;
- specify who stores/refreshes provider tokens;
- verify agent/connection lifecycle and evidence;
- never equate generic RFC 8693 support with Okta `oauth-sts` parity without an interoperability test.

The default solution for this phase is the hybrid MCP Bridge path.

## Phase 11 — Teardown and retention

Delete synthetic users/groups, virtual keys, WLP signing credentials, Resource Connections, model credentials, test tokens/caches, and the intentionally less-governed Phase 1 control environment. Retain only approved sanitized evidence for the declared period, verify deletion from backups according to policy, and record teardown ownership and completion.

Gate: no active demo credential or unintended public endpoint remains, and the retention inventory matches the evidence store.

## Proposed implementation repository shape

Create these only after the plan is approved:

```text
src/okta_litellm/        # thin auth, identity, policy, and evidence integration
litellm/config.yaml      # secret references only
patches/                 # isolated upstream-compatible patch, if required
demo/claude/             # client configuration template
demo/swiss-army-mcp/     # synthetic resource configuration
infra/                   # local/demo deployment manifests
tests/unit/
tests/integration/
tests/e2e/
vendor/README.md         # exact LiteLLM tag, commit, image, digest, provenance
```

Do not vendor the LiteLLM source tree or copy raw Postman collections into the repository.
