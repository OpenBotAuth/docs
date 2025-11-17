# Overview

This sequence diagram shows:

<figure><img src="../.gitbook/assets/mermaid-diagram-2025-11-16-175938.png" alt=""><figcaption></figcaption></figure>

* **Registration Phase**: Bot operator publishes signature-agent card to OpenBotRegistry
* **Request Phase**: Bot signs HTTP request with private key; origin server queries OpenBotRegistry directly for the signature-agent card to get public key
* **Verification**: Origin server validates signature using public key from agent card, then policy engine evaluates intent and scopes
