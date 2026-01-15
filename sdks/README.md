# SDKs Overview

OpenBotAuth provides official SDKs for integrating signature verification directly into your applications. Both SDKs follow the same design principles:

- **No local cryptography** - All verification is delegated to the OpenBotAuth verifier service
- **Middleware support** - Drop-in middleware for popular frameworks
- **Observe mode** - Non-blocking verification for gradual rollout
- **Security-first** - Sensitive headers are never forwarded

## Available SDKs

| SDK | Package | Frameworks |
|-----|---------|------------|
| [Node.js / TypeScript](nodejs.md) | [@openbotauth/verifier-client](https://www.npmjs.com/package/@openbotauth/verifier-client) | Express, Next.js |
| [Python](python.md) | [openbotauth-verifier](https://pypi.org/project/openbotauth-verifier/) | FastAPI, Flask, Starlette |

## When to Use SDKs

Use the SDK approach when:

- You want fine-grained control over verification logic
- You need to integrate with your existing authentication/authorization
- You want to make policy decisions in your application code
- You're building a Node.js or Python application

## Alternative: Proxy

If you prefer a zero-code approach or use a different language/framework, consider the [OpenBotAuth Proxy](../proxy/README.md) which sits in front of any HTTP backend.

## Core Concepts

### Verification Flow

```
1. Extract RFC 9421 signature headers from request
2. Send headers to verifier service
3. Receive verification result (verified, agent info, or error)
4. Make policy decision in your application
```

### Middleware Modes

Both SDKs support two middleware modes:

| Mode | Behavior |
|------|----------|
| `observe` (default) | All requests pass through; verification result attached to request |
| `require-verified` | Protected paths return 401 if verification fails |

### Request State

After middleware processing, verification state is attached to the request:

```javascript
// Node.js
req.oba.signed    // boolean - request had signature headers
req.oba.result    // VerificationResult or null
```

```python
# Python
request.state.oba.signed    # bool - request had signature headers
request.state.oba.result    # VerificationResult or None
```
