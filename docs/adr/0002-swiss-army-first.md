# ADR 0002: Use Swiss Army MCP before GitHub

- Status: Proposed
- Date: 2026-09-11

## Context

The core question is whether Okta can govern human-plus-agent access while LiteLLM enforces at the MCP gateway. An internal resource behind an Okta Custom AS can expose the final identity and scope claims directly. GitHub instead requires provider OAuth and Okta STS brokered consent, adding a separate interoperability question.

## Decision

Use a synthetic, read-only Swiss Army MCP tool as the primary demonstration. Add a mutable simulation only after the read and denial gates pass. Treat GitHub as a secondary hybrid or later STS scenario.

## Consequences

- Token claims, scope checks, side effects, and failure behavior are deterministic.
- The resource can reject direct bypass and wrong-audience tokens visibly.
- The demo proves XAA without confusing it with provider consent.
- It does not prove third-party SaaS STS/OPA parity; that limitation must remain explicit.
