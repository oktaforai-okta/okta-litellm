# Contributing

Start with [BUILD-START-HERE.md](BUILD-START-HERE.md) and follow [AGENTS.md](AGENTS.md). This repository is planning-first; runtime work must reference an approved implementation phase and its validation gate.

## Pull request requirements

- State the selected topology, phase, environment class, and acceptance gates affected.
- Include tests/evidence for behavior changes and update architecture/security documentation when trust boundaries change.
- Use synthetic examples and placeholders; never include tenant/customer identifiers or credentials.
- Pin external dependencies/images and include provenance, SBOM, vulnerability, and license results when implementation begins.
- Keep LiteLLM upstream changes isolated and upstream-compatible where possible.
- Require review from the owners of identity/security and the affected gateway/resource.

## MCP Bridge boundary

MCP Bridge is proprietary. Do not contribute its source, copied implementation, internal tests/comments, image contents, private repository links, or reconstructed schemas. Original API/setup documentation and externally observable behavior are permitted. Where approved detailed documentation is unavailable, record the gap and request it from the Bridge owner.

## Commit hygiene

Before committing, scan for tokens, JWTs, private keys, secrets, Postman environments, packet captures, local state, logs, and customer data. Keep generated/runtime state ignored. Use focused commits and do not mix unrelated local changes.

