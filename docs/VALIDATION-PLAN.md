# Validation plan

## Release and edition gates

| ID | Test | Pass condition |
|---|---|---|
| V-01 | Verify image signature and digest | Artifact resolves to approved stable source and immutable digest |
| V-02 | Feature/license inventory | Every configured feature is present in the chosen edition |
| V-03 | Restart durability | User mapping and encrypted assertion survive restart without leaking secrets |
| V-04 | SBOM/vulnerability/license scan | Approved findings, licenses, provenance, and exception owner are recorded |
| V-05 | Configuration reconciliation | O4AA/LiteLLM/resource mapping ledger has no stale, duplicate, or unknown mapping |
| V-06 | Reference environment reproduction | Clean environment deploys from pinned artifacts with the documented trust boundaries |
| V-07 | Build intake | User-confirmed execution brief answers all nine questions without containing secrets |

## Identity gates

| ID | Test | Pass condition |
|---|---|---|
| I-01 | Decode final resource token safely | Expected `iss`, `aud`, human subject, intended logical-agent actor in the frozen tenant-issued claim profile, scope, `jti`, `iat`, and `exp`; no fabricated claim |
| I-02 | Wrong assertion issuer/subject | Okta denies; no resource request |
| I-03 | Caller changes agent header | Actor is unchanged or request is denied |
| I-04 | Shared route attempted by a second agent | Denied or isolated; never attributed to Claude silently |
| I-05 | Direct bearer with no stored SSO assertion | Behavior is known; demo bootstrap prevents list/call subject mismatch |
| I-06 | User A LiteLLM key + User B valid Org-AS ID token | Denied because admission owner and token subject differ; otherwise direct-token mode is prohibited |
| I-07 | Copy User A credential to raw client/other host | Denied by sender/session binding, or result is explicitly classified as route attribution rather than Claude runtime attestation |
| I-08 | Concurrent User A/User B list and call | No assertion, key, cache, response, or evidence crossover |
| I-09 | Logout, suspension, key/session revocation, re-login | Fresh work follows the documented revocation and reauthentication boundary |
| I-10 | OIDC app linkage/assignment and WLP/key activation | Missing/unlinked/inactive state denies fresh issuance |
| I-11 | Claude and custom UI operate concurrently | Distinct WLPs/routes/credentials; no assertion, cache, response, or evidence crossover |
| I-12 | Custom UI OAuth | Authorization code with PKCE succeeds for its own client; no Bridge admin or downstream token reaches the browser |
| I-13 | Identity-like MCP arguments | Forged subject, agent, scope, approval, or correlation fields are ignored/rejected and never override transport/token identity |
| I-14 | Human-only OBO token/profile | Rejected for the governed path, or the result is explicitly disqualified from the first-class-agent claim |
| I-15 | Normal developer workflow | Allowed user completes standard login/consent without manually moving tokens; warm calls require no unexplained identity step |

Do not hard-code an assumed WLP claim location before observing the tenant. Record the exact final-token profile, then make the resource validator enforce that profile.

## Authorization and enforcement gates

| ID | Test | Expected result |
|---|---|---|
| A-01 | Eligible user, active agent, valid read scope | Success |
| A-02 | Ineligible user | Okta grant denial; no upstream side effect |
| A-03 | Resource Connection removed/inactive | Okta grant denial |
| A-04 | Requested scope removed from policy | Okta grant denial |
| A-05 | Unknown LiteLLM key | Admission denial before token exchange |
| A-06 | Key has no explicit MCP grant | Discovery/call denied, not default-open |
| A-07 | Hidden tool called by raw JSON-RPC | Denied even if the UI does not display it |
| A-08 | Final token replayed to wrong audience | Resource denial |
| A-09 | Expired, altered, or insufficient-scope token | Resource denial |
| A-10 | LiteLLM virtual key sent directly to resource | Resource denial |
| A-11 | Direct network call with a valid but unauthorized object | Tool/FGA denial |
| A-12 | Valid authorized resource bearer replayed from untrusted host | Network/mTLS/sender constraint denies; otherwise mandatory-LiteLLM claim fails |
| A-13 | Read allowed, simulated write denied on same MCP server | Enforcement layer and scope/tool limitation are unambiguous; no side effect |
| A-14 | Wrong/missing `audience` or `id_jag_resource` | Exchange or resource rejects; no ambiguous default succeeds |
| A-15 | Direct model-provider configuration | Blocked in demo profile and recorded as bypass test |
| A-16 | Direct MCP-resource configuration | Blocked in demo profile and recorded as bypass test |
| A-17 | Raw inbound auth/identity/forwarding/host headers | Cannot overwrite trusted egress token, subject, agent, destination, or correlation ID |
| A-18 | Tool-loop/rate-limit abuse | Per-user/route budgets stop the loop without identity crossover or partial mutable side effects |
| A-19 | Mixed allowed/disallowed scope request | Exact requested set is recorded; Okta denies or downscopes only according to the verified policy contract |
| A-20 | Missing/wrong WLP, `kid`, issuer, audience, resource indicator, or JWKS | Fails closed before resource side effect; no shared/ephemeral credential fallback |
| A-21 | Protected and unprotected tools share a client profile | Release gate fails; governed profile contains only the intended protected route |
| A-22 | Unapproved model alias/provider requested | LiteLLM denies it; control and governed profiles do not use master keys or unrestricted wildcard routes |
| A-23 | Same agent/resource, eligible versus ineligible human | Eligible call succeeds and ineligible call is denied; agent/gateway authority never lifts the human entitlement ceiling |
| A-24 | LiteLLM admission key with no Okta resource token | Swiss Army denies it; the key cannot serve as the last-mile resource credential |
| A-25 | More-privileged agent route with under-entitled human | Okta/resource denies capabilities above the human ceiling; no side effect |
| A-26 | Optional own-object versus other-object fixture | Resource/FGA permits only the authorized object and records the object-level decision separately from OAuth scope |

For the native MVP, Okta's authoritative decision is resource/scope token issuance. LiteLLM's local route/tool rules are a coarse enforcement ceiling. Do not claim dynamic per-tool Okta policy until the tool-to-scope/list-hook extension is implemented and separately tested.

## Custody gates

| ID | Test | Pass condition |
|---|---|---|
| C-01 | Successful call evidence | Human, agent, target, tool, scope, grant, and outcome reconstructable |
| C-02 | Denied call evidence | Denial stage/reason and absence of resource side effect shown |
| C-03 | Secret scan | No bearer, ID-JAG, refresh token, key, secret, or sensitive raw argument |
| C-04 | Correlation | LiteLLM and resource events join by a trusted request ID; token issuance joins through hashed `jti` and Okta event/transaction ID where exposed |
| C-05 | Log integrity | Events exported to controlled append-only/SIEM destination for the demo |
| C-06 | Concurrent collision test | Identical, delayed, duplicated, and reordered calls reconstruct without an ambiguous or incorrect join |
| C-07 | “Why” evidence | Sanitized task/reason ID, session capture time, key owner, and policy/config revision are retained without prompts or secrets |
| C-08 | Evidence completeness/retention | Required fields, maximum lag, deletion date, access audit, and missing-event behavior pass |
| C-09 | Invalid/unverified JWT presented | No successful identity/custody classification; diagnostic data is clearly untrusted and excluded from evidence |
| C-10 | Claim minimization | Evidence contains only approved claim fields, not the raw token or whole claims object |

LiteLLM's spend/action logs are operational evidence; its configuration audit table is not by itself a tamper-evident decision ledger. The evidence design must add a sanitized action receipt.

## Lifecycle gates

| ID | Test | Pass condition |
|---|---|---|
| L-01 | Deactivate agent | Forced fresh exchange fails; already-issued bearer lifetime is measured separately |
| L-02 | Disable connection | Fresh exchange fails |
| L-03 | Remove user eligibility | Fresh exchange fails |
| L-04 | Rotate signing key | New key succeeds; retired key fails after declared overlap |
| L-05 | Rotate LiteLLM encryption/master key | Stored assertion remains readable through supported rotation process |
| L-06 | Warm-cache deactivation without eviction | Last successful gateway/direct replay is measured and stays within stale-authorization SLO |
| L-07 | Revoke Claude admission credential/session | Fresh admission fails; other users and model/MCP credentials remain isolated |
| L-08 | User suspension and group removal | Fresh issuance fails after measured cache window |
| L-09 | Missing/malformed leg-two `expires_in` | Effective cache TTL never exceeds approved maximum; unsafe default fails the gate |

## Resilience gates

| Fault | Required behavior |
|---|---|
| Okta token endpoint timeout | MCP call fails closed; model-only routing may continue independently |
| Malformed/deny token response | No upstream MCP request and no weaker credential fallback |
| Missing/expired stored assertion | Clear 412/reauthentication path, not anonymous execution |
| Identity assertion store unavailable | 503/fail closed |
| Resource 401 | Invalidate cached token, re-mint once, retry at most once |
| Resource 403 | Surface denial; do not retry authentication |
| Evidence sink unavailable | Retain a minimal durable receipt or fail the sensitive demo action |
| LiteLLM restart | No identity crossover; durable state restores safely |
| MCP Bridge fallback selected | Manual mode switch and fresh Bridge authentication; never automatic bypass |
| Model provider unavailable | Model call fails on its own plane; no MCP credential fallback or authorization weakening |
| DNS rebinding/SSRF target supplied as MCP URL | Destination allowlist and egress controls reject it |
| Prompt/tool metadata injection | Untrusted descriptions/results cannot change route, credentials, policy, or approval state |
| Downstream MCP session becomes stale after resource restart | Invalidate session and retry at most once; preserve exact identity, credential, scopes, resource, and topology |
| Custody callback/viewer misclassifies a known fixture | Authoritative Okta/gateway/resource events win; projected record is rejected and alerted |
| Resource returns an application-level denial | Gateway/evidence records denial, not successful tool execution; no side effect exists |

## Non-functional requirements

The quantitative gates, measurement method, operational ownership, recovery targets, and data lifecycle are defined in [NFR-AND-OPERATIONS.md](NFR-AND-OPERATIONS.md). They must be approved before deployment; unmeasured adjectives such as "fast" or "highly available" are not pass criteria.

- No long-lived access token reaches Claude or appears in logs.
- Private signing keys remain non-exported from the deployment secret boundary where feasible.
- Token and decision caches key at least on tenant, human, agent, resource, scope/action, and credential generation.
- Denials and errors are never converted into cached allows.
- All clocks are synchronized; JWT validation has a small documented skew allowance.
- Demo latency records each token leg separately and states whether a cache hit occurred.
- Protected MCP availability must not determine model-routing availability.
- Synthetic mutable actions are idempotent and have verifiable side-effect IDs.
- Model requests and tool inputs/results are classified for provider egress; sensitive tool output is not sent to a model unless explicitly approved.
- Database dumps, replicas, backups, debug/error logs, callbacks, crash traces, and restored environments pass secret-leakage tests.

## Exit criteria

Ship the customer demo only when every MVP row marked above passes on the pinned artifact and tenant. Record known deviations next to the evidence; do not waive identity, bypass, resource validation, or secret-handling gates.
