# Open questions and decision gates

These questions are intentionally unresolved by desk research. The implementation spike must answer them with tenant and wire evidence.

| Priority | Question | How to resolve / evidence artifact | Accountable role | Due / blocks |
|---|---|---|---|---|
| P0 | Does the tenant accept the O4AA WLP identifier in LiteLLM's configured `client_id` path? | Sanitized leg-one trace, Okta event, final actor claim | Okta/O4AA owner | Gate 0B |
| P0 | What exact tenant-issued claim profile identifies the human and logical agent in the final Custom-AS token and System Log? | Sanitized token/event profile + resource validation test; no fabricated custom actor claim | Okta/O4AA + resource owners | Gate 2 |
| P0 | Can the intended O4AA agent and linked OIDC app both be assigned to the required policies? | Object/status manifest and allow/deny evidence | Okta/O4AA owner | Gate 2 |
| P0 | Which LiteLLM features require an enterprise license in the customer's edition? | Signed capability/license ledger for pinned image | LiteLLM owner | Gate 0A |
| P0 | Do `tools/list` and `tools/call` use the same stored human subject on the exact route? | Generic-OIDC, restart, concurrent-user JSON-RPC report | Identity + test owners | Gate 4 |
| P0 | Can an inbound User B ID token be combined with User A's LiteLLM key? | Cross-user substitution test; otherwise disable direct-token mode | Identity + security owners | Gate 4 |
| P0 | What token/cache TTL bounds the deactivation story, including invalid `expires_in`? | Warm-cache trace through last success and fresh denial | LiteLLM + security owners | Gate 7 |
| P0 | Can a valid resource bearer bypass LiteLLM from an untrusted host? | Network/mTLS/sender-constrained replay report | Infrastructure + resource owners | Gate 5 |
| P1 | What Swiss Army MCP version, tools, scopes, issuer, audience, resource indicator, and claim profile will be used? | Frozen synthetic resource contract | Resource owner | Gate 2 |
| P1 | Can a trusted correlation ID reach Swiss Army, and which `jti`/Okta event field is joinable? | Concurrent/reordered custody reconstruction report | Observability owner | Before Phase 3 |
| P1 | Is one dedicated LiteLLM deployment required, or is one isolated route sufficient? | Cross-key/cross-agent isolation report | LiteLLM + architecture owners | Gate 4 |
| P1 | Does the customer require per-tool Okta decisions or is resource/scope issuance sufficient? | Approved customer claim/requirements record | Product owner | Before customer demo |
| P1 | Does one policy rule need to cover the complete requested scope set, and is partial downscoping supported? | Wire tests for allowed-only, disallowed-only, and mixed scope requests | Okta/O4AA owner | Gate 5 |
| P1 | What is acceptable behavior during Okta/evidence outages? | Approved SLO and failure-policy record | Security + service owners | Before Phase 3 |
| P1 | Which deployment target, data region, model provider, and retention policy are approved? | Trust-boundary diagram and data inventory | Architecture + privacy owners | Gate 0A |
| P2 | Is GitHub required in the first customer session? | Approved demo scope | Product owner | Before Phase 8 rehearsal |
| P2 | Should the thin integration be upstreamed to LiteLLM or maintained separately? | Maintainer engagement/ownership ADR | Engineering owner | After Gate 9 decision |

No P0 item may be converted into an assumption in customer-facing material.
