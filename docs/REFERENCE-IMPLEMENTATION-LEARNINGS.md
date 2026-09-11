# Comparative implementation learnings

Reviewed 2026-09-11 from a private demonstration repository supplied by a collaborator. This document records architectural observations and transferable lessons only. It does not copy that repository's source, credentials, tenant identifiers, private links, or proprietary MCP Bridge implementation.

## What the comparison actually proved

The reference implementation used three narrative states:

| State | Observed path | What it demonstrated |
|---|---|---|
| Ungoverned | Claude Code → LiteLLM model/MCP gateway → intentionally unprotected toy MCP resource | LiteLLM virtual-key routing and logging without an Okta-governed human/resource authorization chain |
| Machine-only | M2M token → LiteLLM | A machine identity is not the same as human-delegated agent work |
| Governed | Claude Code → MCP Bridge → Okta XAA/ID-JAG → token-validating MCP resource | Human-context resource authorization, scope enforcement, and Bridge/Okta/resource evidence |

The governed state did **not** route MCP traffic through LiteLLM. It is therefore evidence for our hybrid fallback's Bridge enforcement pattern, not proof that LiteLLM can replace Bridge as the MCP PEP. It also did not prove that model inference remained routed through LiteLLM in the governed state.

The reference evaluated an older LiteLLM/Bridge combination than this plan. Our selected LiteLLM baseline includes a newer native `oauth2_id_jag` path and stored generic-OIDC assertion support, so its earlier feasibility conclusion is informative but not authoritative. Re-run every gate on the pinned versions.

## Lessons adopted into this plan

### 1. Let the issued token profile speak for itself

The reference initially expected an `act` claim but observed a tenant/Bridge flow where human and client/agent attribution appeared through `sub` and `cid`, while richer actor structure was visible in Okta's System Log. The durable rule is:

- record the exact tenant-issued token and System Log profile;
- validate the intended human and logical agent using documented claims for that profile;
- never add a custom/fabricated `act` claim merely to make the demo narrative pass;
- fail the identity gate if the intended logical agent cannot be proven.

This complements, rather than weakens, the O4AA requirement. Claim names are not the security outcome; verified human-plus-agent attribution is.

### 2. Identity must come from a trusted transport boundary

The protected resource must derive identity from the validated bearer and trusted gateway context. Human subject, agent ID, scope, audience, correlation ID, or approval state must never be accepted from MCP tool arguments or model-generated content. Add a negative call that supplies forged identity-like arguments and prove they are ignored or rejected.

### 3. Separate protected and intentionally unprotected client profiles

When two MCP servers expose similarly named tools, the model may choose the unintended route and return a plausible result for the wrong security path. Use separate Claude project profiles/configurations for the no-Okta control and the governed demo. The custom UI must expose only its selected topology. Log and display the actual gateway, MCP server, namespaced tool, and resource-side side-effect ID for every demonstration call.

### 4. Tool visibility is not authorization

The reference needed explicit LiteLLM virtual-key MCP permissions before calls worked, while the Bridge could retain or cache tool metadata separately from per-call resource authorization. Test discovery and invocation independently. A successful `tools/list` never proves the user can execute the tool.

### 5. Requested scopes may be an all-or-nothing policy input

A policy may require one rule to cover the complete requested scope set rather than partially granting it. Test the exact wire scope set, including a request containing both allowed and disallowed scopes. Do not assume automatic downscoping. Prefer a purpose-bound agent/route requesting one least-privilege scope set over a live scope flip.

### 6. A live scope flip is operationally fragile

The reference changed one Resource Connection scope between personas and had to synchronize/reset it after every run. That is a useful policy-change illustration but a poor primary authorization design. This plan keeps distinct logical agents/routes and deterministic fixtures. If a scope flip is shown, record the starting revision, force and verify synchronization, invalidate/wait out caches as documented, then restore and re-read the baseline.

### 7. Key identity and issuer configuration must fail closed

The signing JWK `kid` must match the active key registered for the intended workload principal. Missing WLP/key configuration must never fall back to an ephemeral or shared gateway identity. At resource startup, assert the exact issuer, audience, resource indicator, JWKS origin, and accepted actor profile so an accidental default authorization server cannot silently invalidate the proof.

### 8. MCP sessions can outlive a resource process

Restarting a downstream MCP resource can leave a cached gateway session stale. Test the deployed gateway's “session not found” behavior, bounded retry, session invalidation, and identity/scope preservation. A retry must not switch resource, credential, user, agent, or topology.

### 9. A callback is not the authorization boundary

The reference's LiteLLM callback initially inspected the wrong credential location, showing that a polished custody view can be wrong even when requests succeed. The authoritative evidence comes from validated Okta issuance, gateway enforcement, and resource-side token/action validation. Callback or UI projections must be derived, sanitized, versioned, and tested against known concurrent calls.

### 10. Demo evidence must not overstate correlation

The reference honestly fell back to time-plus-business-object joins because its systems lacked a common correlation ID. Our customer claim requires a trusted request ID across gateway/resource plus token issuance linkage through hashed `jti` and an Okta event/transaction identifier where available. Ambiguous events stay unlinked; they are never guessed into a chain.

### 11. Preserve LiteLLM's demonstrated value in every governed run

The comparison clearly showed LiteLLM's model routing, budgets, virtual keys, and MCP allowlists as valuable but distinct from Okta entitlement decisions. Our governed run must prove the model request still traverses LiteLLM even when MCP enforcement moves to Bridge in the hybrid topology.

### 12. Presentation code must never upgrade unverified data into evidence

A demo callback may be tempted to decode an invalid JWT without signature verification so the screen remains populated. That is acceptable only as clearly labeled diagnostic text and is forbidden for custody classification or authorization. Evidence must come from successfully verified tokens/events, use an allowlisted claim projection rather than an entire claims object, and record verification outcome and validator profile.

Likewise, a tool that returns an “access denied” string inside an otherwise successful MCP result can be misclassified as success. Denials need a machine-readable error/outcome, no side effect, and matching gateway/resource audit state.

### 13. Avoid demo shortcuts in the control plane

The intentionally ungoverned comparison still should not use LiteLLM's master/admin key, wildcard model routing, unpinned dependencies, or mutable usernames as authorization identifiers. Use a least-privilege virtual key, approved model aliases, locked dependencies, and immutable `iss` + `sub` mapped server-side to the synthetic business record. This preserves a fair comparison: the missing control is O4AA governance, not basic demo hygiene.

## What not to copy

- Do not copy its toy-resource code, callback, scripts, local ports, credentials, tenant objects, private repository structure, or MCP Bridge source/deployment.
- Do not repeat broad statements such as “LiteLLM has no identity” or “there are no logs.” State the missing Okta-governed controls precisely.
- Do not use `sub != cid` or any single claim heuristic without first freezing the target tenant's issued-token contract.
- Do not use an M2M token as a substitute for a human-delegated agent flow.
- Do not rely on a scope flip, warm tool catalog, gateway callback, or time-window join as the sole proof of authorization.
- Do not infer current LiteLLM licensing/capability from the older evaluated release; use the Phase 0 edition and stable-version gate.

## Additional validation gates derived from the comparison

1. Protected/unprotected MCP profiles cannot coexist in the same demo client configuration.
2. Forged identity, scope, agent, correlation, or approval fields in tool arguments have no authorization effect.
3. The complete requested scope set is captured and an allowed-plus-disallowed request is denied or downscoped only according to verified Okta behavior.
4. Missing/wrong WLP, `kid`, issuer, audience, resource indicator, or JWKS origin fails before a protected side effect.
5. A stale downstream MCP session is invalidated and retried at most once without changing identity or authorization.
6. Both Claude and the custom UI show model-plane evidence from LiteLLM during the governed run.
7. Projected custody output is reconciled against authoritative raw-system events using synthetic concurrent fixtures.
8. Invalid/unverified JWTs and whole token-claim objects cannot populate a successful custody record.
9. A resource denial is machine-readable and cannot be counted as a successful tool call.
10. The no-Okta control uses a least-privilege virtual key and approved model aliases, never a master key or unrestricted wildcard route.
