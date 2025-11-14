---
description: ELI5 for Open Bot Auth
---

# TL;DR

An easy website plugin that lets you authenticate bots and charge them for accessing your content.

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

### Why is this cool?&#x20;

Because you get to define custom pricing and payment terms without relying on CDN providers as payment intermediaries.

Refer to [Usecases](usecases.md) to read more on different industry use cases.&#x20;

