---
description: ELI5 for Open Bot Auth
---

# TL;DR

OpenBotAuth proves your ownership of the agent to the website and allows your agent to pay on your behalf for consuming content. Once you create your profile on the Registry, you can create agent certificates tied to your identity.&#x20;

OpenBotRegistry is an attempt to tie your already verified social idenitites to agents, making verification process fast and accessible.&#x20;

### Why is this cool?&#x20;

1. Developers don't have to keep buying websites to hosts JWKS (or[ Signature Agent Cards](https://thibmeu.github.io/http-message-signatures-directory/draft-meunier-webbotauth-registry.html#name-signature-agent-card)) or worry about tracking key rotation.
2. Identities are not tied to a CDN, ie, identities are open to the whole  web, if they choose to adopt it.&#x20;
3. Websites transact directly with the agent owner rather than middlemen. Content creators maintain control over pricing and access rights.&#x20;

<figure><img src=".gitbook/assets/TLDR (1).png" alt=""><figcaption></figcaption></figure>

### What is an "Origin"?

The **origin** is your actual website server.

### What is a "Proxy"?

A **proxy** is a middleman  that sits between bots and your origin server. Instead of visitors connecting directly to your website, they connect to the proxy first, and the proxy forwards requests to your origin.

### What is Direct Verification? <a href="#what-is-direct-verification" id="what-is-direct-verification"></a>

**Direct verification** means your origin server checks if a bot is legitimate without relying on a proxy or middleman.​​

Here's how it works:

1. The bot creates a digital signature.
2. The bot sends this signature directly to your origin server (example.com) along with their request (we call this intent).​​
3. Your origin server verifies the signature​

Refer to [Usecases](usecases.md) to read more on different industry use cases.&#x20;

