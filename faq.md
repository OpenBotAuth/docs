---
description: To add questions in this section, please open a Github Issue.
---

# FAQ

#### Why Web Bot Auth? Not OAuth?

Oauth introduces complexity in implementation. There's an open issue on Thibault's [Github](https://github.com/thibmeu/http-message-signatures-directory/issues/13) discussing this. There another cloudflare project, [OpenPubkey](https://www.bastionzero.com/openpubkey) which works with OIDC and they use an MPC. OpenBotAuth intends to avoid any complex cryptography.&#x20;

#### Why Can't I Sign-in with Facebook/X?

Agents live and die in developer's terminal. Almost all developers have a Github Account and understand the value of hosted services. Developers remain the primary agent builders. Hence it was a no brainer to allow Github Profiles to verify first. We will consider expanding to other verification methods.

