# Gap audit

Final planning audit date: 2026-09-11. Three independent passes examined product/protocol behavior, architecture/operations, and adversarial claim falsification. The core LiteLLM-first recommendation remains conditional; the following corrections are part of the plan and are not optional polish.

## Non-waivable gaps and disposition

| Priority | Gap that could create a false customer claim | Planning disposition |
|---|---|---|
| P0 | A dedicated route/WLP identifies a configured logical agent but does not attest the Claude binary; a copied key may impersonate the route | Narrowed the identity claim; added short-lived user/route binding, raw-client/cross-host replay test, and proof-of-possession/attestation decision gate |
| P0 | User A's LiteLLM key could be paired with User B's inbound ID token | Stored-assertion-only is the primary path; direct-token mode is prohibited until the cross-user substitution test passes |
| P0 | Claude's model path and MCP path were conflated | Added separate model and MCP topology, configuration, telemetry, bypass, availability, and correlation gates |
| P0 | A valid resource token could bypass LiteLLM | Made private/source-restricted Swiss Army ingress a demo gate and added valid-bearer replay testing; otherwise the mandatory-PEP claim is downgraded |
| P0 | Time-window correlation could join the wrong concurrent call | Required a trusted gateway correlation ID plus hashed `jti`/Okta event join; time proximity is contextual only |
| P0 | O4AA app linkage, assignment, activation, and exact target parameters were incomplete | Added explicit User Sign-on linkage/status checks and frozen `audience`/`id_jag_resource` mappings with negative tests |
| P0 | Cache eviction could hide the true deactivation delay | Added warm-cache/no-intervention measurement, unsafe `expires_in` handling, explicit stale-access SLO, and separate fresh-issuance proof |

## Execution-readiness gaps and disposition

| Priority | Gap | Planning disposition |
|---|---|---|
| P1 | Client bootstrap, session/key lifecycle, and concurrent-user isolation were ambiguous | Defined generic OIDC → immutable user mapping → short-lived route-bound credential and lifecycle/concurrency tests |
| P1 | One safe tool could overstate dynamic Okta tool authorization | Added read-versus-simulated-write negative case and claim language separating Okta resource scopes, LiteLLM local ceilings, and resource authorization |
| P1 | Resource Connections and LiteLLM permissions can drift | Added an owned mapping ledger, pre-demo reconciliation, alerting, and fail-closed unknown mappings |
| P1 | Stored assertions, keys, logs, backups, and model-provider egress lacked a lifecycle | Added data classification, residency, encryption/KMS separation, retention/deletion, backup/restore, access, rotation, offboarding, and leakage gates |
| P1 | Prompt injection, SSRF, header collision, cross-user cache leakage, admin access, bypass, and tool-loop abuse were missing threats | Added mitigations and falsification tests to the threat and validation plans |
| P1 | NFRs, observability, support, recovery, and ownership were qualitative | Added proposed measurable targets, required telemetry/runbooks, RACI, and environment/promotion controls |
| P1 | Hybrid fallback was architectural, not executable | Added pre-provisioning, manual endpoint switch, fresh authentication, validation, rehearsal, and RTO; automatic denial bypass remains forbidden |
| P1 | Supply chain, cleanup, and handoff were incomplete | Added SBOM/scans/license/provenance, support handoff, environment teardown, credential deletion, and retention verification |
| P1 | Prior SSO could be described as fresh human authorization | Changed evidence/presenter language to “delegated session and permissions”; step-up/HITL is required for a per-action presence claim |

## Residual go/no-go decisions

The plan is ready to execute only after named owners accept the targets. It is not yet proof that the integration works. The build must stop or select MCP Bridge for the MCP plane if any of these remain true:

1. Okta does not issue the intended human-plus-logical-agent token from LiteLLM's configured WLP credential.
2. User, route, credential, assertion, and cache binding cannot prevent cross-user substitution.
3. Swiss Army cannot validate the exact issuer/audience/scope/actor profile or cannot be protected from untrusted direct access.
4. Custody events cannot be joined unambiguously under concurrency.
5. Fresh authorization cannot be stopped within the approved stale-access window.
6. Required LiteLLM capabilities are unavailable in the customer's edition or pinned stable artifact.
7. The data/privacy, recovery, or operational controls are unacceptable for the customer's deployment.

Passing the narrow demo does not prove multi-agent credential selection, per-tool dynamic Okta policy, GitHub/Atlassian STS, OPA vaulted secrets, automatic Resource Connection synchronization, or full MCP Bridge replacement.
