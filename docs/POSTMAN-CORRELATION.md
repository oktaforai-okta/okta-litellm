# Postman collection correlation

The source folder contained 18 Okta flow collections plus an Atlassian collection and two environments. They are useful protocol specimens, not production assets. No collection or environment has been copied into this repository.

## The matrix

| Context | OPA | STS | XAA |
|---|---|---|---|
| Human identity (HI) | 01–03 | 04–06 | 07–09 |
| Non-human identity (NHI) | 10–12 | 13–15, deliberately `_X_` | 16–18 |

Each group then varies client authentication:

1. client ID/public client;
2. client secret;
3. public/private key (`private_key_jwt`).

The collections solve two independent questions:

- **Who is the subject?** Human flows begin with authorization code + PKCE and obtain a user token; NHI flows begin with client credentials.
- **How does the agent reach the resource?** OPA retrieves a vaulted secret, STS brokers a provider OAuth token/consent relationship, and XAA produces a user-plus-agent resource token through ID-JAG.

## Direct mapping to LiteLLM

| Collection behavior | LiteLLM correlation | Planning consequence |
|---|---|---|
| HI XAA, especially 09 private-key | Native `oauth2_id_jag` two-leg flow | Canonical wire oracle for the MVP |
| NHI XAA 16–18 | Potential machine-context/A2A flow | Not the first demo; subject-token eligibility must be proven |
| HI STS 04–06 | Okta brokered provider OAuth/consent | Generic token exchange is not automatically equivalent |
| `_X_` NHI STS 13–15 | Shows the human consent mismatch | Do not treat STS as a pure client-credentials flow |
| HI/NHI OPA | Okta-held static secret delivery | Native LiteLLM ID-JAG does not replace it |
| Client secret variants | LiteLLM fallback supports a secret | Do not choose this for the O4AA workload MVP |
| Private-key variants | Agent signs assertion | Preferred pattern, subject to WLP identity gate |

## Canonical MVP comparison

Collection 09 is closest to the target:

```text
Human authorize + PKCE
  -> human token / ID token
  -> Org-AS ID-JAG request signed by agent
  -> Custom-AS JWT bearer exchange
  -> scoped resource access token
```

Compare captured LiteLLM requests by parameter names and semantics, not by copying secrets or static JWTs. LiteLLM's exact leg-two form is not byte-for-byte identical to every collection example, so tenant interoperability remains an acceptance test.

## Quality and security findings

- Nine human token requests reuse one static DPoP proof; a DPoP proof must be freshly generated with correct method, URI, time, and unique ID.
- Some NHI OPA/STS private-key assertions are generated in post-response tests rather than before the request that needs them.
- Collection 17 is labeled client-secret but duplicates the client-ID-only behavior.
- Human examples start against a Custom AS, while the standard O4AA ID-JAG human subject is an Org-AS ID token. The demo must use the Org-AS subject unless a different tenant-supported profile is explicitly proven.
- The Atlassian collection contains literal credential/assertion material and an unresolved `cloudId`. Treat it as compromised demonstration material; never copy or commit it.
- Environment files may contain identifiers or secrets and are explicitly excluded from the future repository.

## Safe use during implementation

1. Build fresh, sanitized requests from protocol requirements.
2. Generate new PKCE, DPoP, `jti`, timestamps, and client assertions for every run.
3. Keep credentials in a secret manager or ephemeral local environment excluded from Git.
4. Capture only headers/claims needed for validation and redact token bodies.
5. Compare expected failures as well as success.
6. If sanitized Postman examples are later desired, reconstruct them from scratch and run automated secret scanning before review.

## Why GitHub is secondary

The GitHub resource is represented by the STS family, not the internal XAA family. Its token is a provider OAuth token obtained through Okta brokered consent; GitHub does not consume the Custom-AS token that proves the Swiss Army scenario. Until LiteLLM reproduces Okta's Resource Connection, interaction/consent, token storage, refresh, and lifecycle semantics, use MCP Bridge for GitHub and keep LiteLLM for model routing.
