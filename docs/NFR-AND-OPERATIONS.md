# NFR and operations plan

These are proposed demo targets, not measured capabilities or customer production commitments. Owners must approve or replace every value before Phase 3, then attach the measurement report to the evidence bundle.

## Measurable demo targets

| Concern | Proposed acceptance target | Measurement |
|---|---|---|
| Concurrency | 20 simultaneous users, 5 MCP calls each, with zero identity/evidence crossover | Mixed-user load test |
| Sustained load | 10 MCP calls/second for 10 minutes; no unbounded queue/cache growth | Gateway/resource metrics |
| Cached MCP latency | p95 ≤ 1.0 s and p99 ≤ 2.0 s excluding tool execution | Correlated gateway spans |
| Uncached token overhead | Record p50/p95/p99 for each Okta leg; proposed combined p95 ≤ 3.0 s | Token-leg spans, cache disabled/missed |
| Timeout/retry | One retry only for an upstream 401 after safe invalidation; no auth retry for 403; total protected-call deadline ≤ 15 s | Fault injection |
| Stale authorization | Proposed maximum 5 minutes after agent/user/connection policy change; actual token/cache limits may require a stricter config or an explicit demo exception | Warm-cache lifecycle test |
| Evidence completeness | 100% of protected calls have a terminal receipt; zero incorrect/ambiguous joins | Concurrent/reordered reconciliation |
| Evidence lag | p95 ≤ 60 s to the demo evidence view | Event timestamps |
| Demo service objective | 99% successful eligible read calls during a 60-minute rehearsal, excluding approved model-provider failures | Synthetic run |
| Recovery | RTO ≤ 30 minutes for restore or manual hybrid switch; RPO ≤ 5 minutes for configuration/evidence, with assertion/key recovery tested separately | Timed exercise |
| Rate limiting | Explicit per-user, per-route, model, and tool-call limits; a loop terminates before budget or 100 calls, whichever comes first | Abuse test |
| Cost | Record model, Okta, compute, storage, and evidence cost for one rehearsal and projected customer scale | Tagged usage report |

If a target is inappropriate for the selected deployment, replace it through an approved decision record rather than silently waiving it. A laptop-only proof must state that availability, RPO/RTO, and horizontal-scale results are non-production limitations.

## Required telemetry

- Model plane: request ID, route/model/provider, latency, token use, budget/guardrail result, and sanitized failure class.
- MCP plane: request ID, human/agent/route binding, server/tool, token-leg latency/outcome, cache hit and age, upstream status, and side-effect ID.
- Identity: OIDC assertion age/expiry, reauthentication, virtual-key status, WLP/key/connection state, and cross-user lookup failures.
- Dependencies: PostgreSQL/Redis health, secret/KMS access, JWKS refresh age, Okta endpoint health, Swiss Army health, and evidence-sink lag.
- Security: 401/403 rates, audience/scope failures, denied bypass attempts, unexpected destinations, header sanitization failures, and mapping-ledger drift.

Dashboards must separate model and MCP availability. Alerts need a threshold, severity, owner, customer/demo impact, and linked runbook. Never use raw credentials, prompts, token bodies, or sensitive tool results as telemetry labels.

## Minimum runbooks

1. Stored assertion expired or subject mapping missing.
2. Okta leg-one/leg-two timeout, denial spike, or claim-profile drift.
3. Swiss Army 401 versus 403 and safe retry behavior.
4. WLP signing-key and LiteLLM encryption-key rotation.
5. PostgreSQL/Redis outage, restore, and cross-user cache-integrity validation.
6. Evidence sink unavailable or reconciliation incomplete.
7. Resource Connection versus LiteLLM mapping drift.
8. Credential/token exposure and forced revocation.
9. Manual hybrid switch and return to LiteLLM.
10. Complete demo teardown and retention verification.

## Hybrid recovery procedure

MCP Bridge must be pre-provisioned and tested if it is presented as a fallback. A named operator manually changes Claude's MCP endpoint from LiteLLM to MCP Bridge, revokes or clears incompatible client credentials/sessions, performs fresh authentication, verifies one allow and one denial with the expected actor/evidence, and records elapsed time. Model traffic remains on LiteLLM. No failed or denied LiteLLM request is automatically replayed through Bridge.

## Data, backup, and privacy

The data inventory must cover OIDC assertions and any refresh material, LiteLLM admission keys, WLP keys, model-provider credentials, caches, configuration/mapping ledger, prompts/tool inputs/results, and custody evidence. For each item record owner, classification, region/residency, encryption and KMS separation, access roles, TTL, retention/deletion, backup/restore, rotation, and offboarding purge.

Backup leakage testing includes database dumps, replicas, object-store versions, debug/error logs, callback payloads, crash traces, and restored environments. Model-provider review must address which tool data leaves the trust boundary, provider training/retention terms, approved region, and redaction. Only sanitized evidence is retained after teardown.

## RACI

Names must replace these role placeholders at kickoff.

| Workstream | Accountable | Responsible | Consulted |
|---|---|---|---|
| Architecture and customer claim | Solution architect | Technical lead | Product, security |
| Okta/O4AA tenant and policy | Okta tenant owner | O4AA engineer | Identity security |
| LiteLLM deployment and model plane | LiteLLM service owner | Platform engineer | Model governance |
| Swiss Army MCP and object authorization | Resource owner | MCP engineer | Security |
| Infrastructure, network, secrets, recovery | Platform owner | DevOps/SRE | Security |
| Evidence, SIEM, privacy, retention | Security owner | Observability engineer | Privacy/legal |
| Validation and release decision | Test owner | Test engineer | All control owners |
| Demo operation and hybrid rollback | Demo owner | Named operator | Support/on-call |
| Incident response and teardown | Service owner | Platform + security responders | All data owners |

## Environment and promotion controls

Use isolated development, integration, and customer-demo environments with distinct credentials and synthetic fixtures. Promote the same immutable artifact and reviewed configuration revision; never copy live assertion databases or caches between environments. The no-Okta control is separately networked, labeled intentionally less governed, reset deterministically, and destroyed immediately after its sanitized evidence is captured.
