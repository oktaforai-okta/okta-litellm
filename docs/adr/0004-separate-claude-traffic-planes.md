# ADR 0004: Validate Claude model and MCP traffic as separate planes

- Status: Proposed
- Date: 2026-09-11

## Context

Claude Code uses an Anthropic-compatible API for model inference and MCP JSON-RPC endpoints for tools. Pointing one plane at LiteLLM does not route or secure the other. The customer values LiteLLM for model routing as well as its proposed MCP enforcement role.

## Decision

Configure Claude's model endpoint and MCP endpoint to LiteLLM independently. Give each plane its own least-privilege admission credential, route, telemetry, availability result, and bypass test. Correlate a user task across planes only through sanitized request/task identifiers; never reuse model or MCP authorization tokens between them.

Repeat the pattern for the custom agent UI using a separate O4AA workload principal, route, credential, session, and evidence identity. Sharing LiteLLM or the MCP resource must not collapse the two logical agents.

The hybrid fallback changes the MCP endpoint to MCP Bridge while the model endpoint remains LiteLLM.

## Consequences

- The demo proves the customer's existing model-routing value and the new MCP authorization path separately.
- An Okta/MCP failure need not disable model-only use, but it cannot trigger direct tool access.
- A model-provider failure cannot weaken MCP identity or token enforcement.
- Operational dashboards, rate limits, credentials, and evidence must distinguish the two planes.
