---
description: Github for AI Agent Identities
---

# OpenBotRegistry

OpenBotRegistry binds your Github identity to a public key, following [RFC 9421](https://www.rfc-editor.org/rfc/rfc9421) configuration. After logging in with Github's authorization, developers may generate a public key, list their agents in a public directory and log agent HTTP messages.&#x20;

The registry hosts a separate Signature Agent Card/JWKS on a supabase link, unique ot your agent. This way it avoids a DNS check for ownership.&#x20;

The registry operates a trusted model in favor of time to market. It extends the [Signature Agent Card ](https://thibmeu.github.io/http-message-signatures-directory/draft-meunier-webbotauth-registry.html#name-signature-agent-card)to include&#x20;

