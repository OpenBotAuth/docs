---
description: Github for AI Agent Identities
---

# OpenBotRegistry

Developers can host all their agents with unique identities, tied to their Github profile, on the registry and use it to log agent activities in public.

OpenBotRegistry binds your Github identity to a public key, following [RFC 9421](https://www.rfc-editor.org/rfc/rfc9421) configuration. After logging in with Github's authorization, developers may generate a public key, list their agents in a public directory and log agent HTTP messages.&#x20;

The registry hosts a separate Signature Agent Card/JWKS on a dedicated link, unique to your agent. This card is signed by OpenBotRegistry as a certificate.&#x20;

The registry operates a trusted model in favor of time to market. It extends the [Signature Agent Card ](https://thibmeu.github.io/http-message-signatures-directory/draft-meunier-webbotauth-registry.html#name-signature-agent-card)to include verified Github profile details.&#x20;

