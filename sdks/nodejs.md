# Node.js / TypeScript SDK

The official Node.js SDK for OpenBotAuth signature verification.

**Package:** [@openbotauth/verifier-client](https://www.npmjs.com/package/@openbotauth/verifier-client)

## Installation

```bash
# npm
npm install @openbotauth/verifier-client

# pnpm
pnpm add @openbotauth/verifier-client

# yarn
yarn add @openbotauth/verifier-client
```

**Requirements:** Node.js >= 18.0.0

## Quick Start

### Express Middleware

The fastest way to integrate is with the Express middleware:

```typescript
import express from 'express';
import { openBotAuthMiddleware } from '@openbotauth/verifier-client/express';

const app = express();

// Add middleware (observe mode by default)
app.use(openBotAuthMiddleware());

app.get('/api/content', (req, res) => {
  const oba = (req as any).oba;

  if (oba.signed && oba.result?.verified) {
    // Verified bot - full access
    res.json({
      content: 'Full article content...',
      agent: oba.result.agent
    });
  } else {
    // Anonymous or unverified - limited access
    res.json({
      content: 'Article preview...',
      upgrade: 'Sign requests for full access'
    });
  }
});

app.listen(3000);
```

### Next.js App Router

For Next.js Server Components and Route Handlers:

```typescript
// app/api/content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import {
  VerifierClient,
  buildVerifyRequestForNext,
  hasSignatureHeaders
} from '@openbotauth/verifier-client';

const client = new VerifierClient();

export async function GET(request: NextRequest) {
  // Check if request has signature headers
  if (!hasSignatureHeaders(request.headers)) {
    return NextResponse.json({ content: 'Preview only' });
  }

  // Build verification request from Next.js request
  const verifyRequest = buildVerifyRequestForNext(request);
  const result = await client.verify(verifyRequest);

  if (result.verified) {
    return NextResponse.json({
      content: 'Full content',
      agent: result.agent
    });
  }

  return NextResponse.json(
    { error: 'Verification failed', reason: result.error },
    { status: 401 }
  );
}
```

### Direct Client Usage

For custom integrations:

```typescript
import { VerifierClient, VerificationRequest } from '@openbotauth/verifier-client';

const client = new VerifierClient({
  verifierUrl: 'https://verifier.openbotauth.org/verify', // default
  timeoutMs: 5000 // default
});

const request: VerificationRequest = {
  method: 'GET',
  url: 'https://example.com/api/content',
  headers: {
    'host': 'example.com',
    'signature-input': 'sig=("@method" "@target-uri" "host");created=1699900000;keyid="key-1";alg="ed25519"',
    'signature': 'sig=:base64signature...:',
    'signature-agent': 'https://registry.openbotauth.org/jwks/mybot.json'
  }
};

const result = await client.verify(request);

if (result.verified) {
  console.log('Verified agent:', result.agent?.client_name);
  console.log('Key ID:', result.kid);
} else {
  console.log('Verification failed:', result.error);
}
```

## API Reference

### VerifierClient

Main client class for calling the verifier service.

```typescript
interface VerifierClientOptions {
  verifierUrl?: string;   // Default: 'https://verifier.openbotauth.org/verify'
  timeoutMs?: number;     // Default: 5000
}

class VerifierClient {
  constructor(options?: VerifierClientOptions);
  verify(request: VerificationRequest): Promise<VerificationResult>;
}
```

### VerificationRequest

Request object sent to the verifier:

```typescript
interface VerificationRequest {
  method: string;           // HTTP method (GET, POST, etc.)
  url: string;              // Full request URL
  headers: Record<string, string>;  // Request headers
  body?: string;            // Request body (for POST/PUT)
}
```

### VerificationResult

Response from the verifier:

```typescript
interface VerificationResult {
  verified: boolean;        // Whether signature is valid
  agent?: {                 // Agent info (if verified)
    client_name: string;
    client_uri?: string;
    // ... other agent metadata
  };
  kid?: string;             // Key ID used for signing
  jwks_url?: string;        // JWKS URL for the agent
  error?: string;           // Error message (if failed)
  created?: number;         // Signature creation timestamp
  expires?: number;         // Signature expiration timestamp
}
```

### Middleware Options

```typescript
interface MiddlewareOptions {
  verifierUrl?: string;     // Verifier service URL
  mode?: 'observe' | 'require-verified';  // Default: 'observe'
  attachProperty?: string;  // Request property name (default: 'oba')
  timeoutMs?: number;       // Verification timeout
}
```

### OBAState

State attached to requests by middleware:

```typescript
interface OBAState {
  signed: boolean;          // Request had signature headers
  result?: VerificationResult;  // Verification result (null if not signed)
}
```

## Header Utilities

Utility functions for working with RFC 9421 headers:

```typescript
import {
  hasSignatureHeaders,
  parseCoveredHeaders,
  extractForwardedHeaders
} from '@openbotauth/verifier-client';

// Check if request has signature headers
const hasSig = hasSignatureHeaders(request.headers);

// Parse covered headers from Signature-Input
const covered = parseCoveredHeaders(signatureInput);
// Returns: ['@method', '@target-uri', 'host', ...]

// Extract only safe headers for forwarding to verifier
const safeHeaders = extractForwardedHeaders(
  request.headers,
  coveredHeaders
);
```

## Security

### Sensitive Headers

The SDK automatically blocks sensitive headers from being forwarded to the verifier:

- `cookie`
- `authorization`
- `proxy-authorization`
- `www-authenticate`

If a `Signature-Input` references any of these headers, the request will be rejected.

### Timeout Handling

All verification requests have a configurable timeout (default 5 seconds). On timeout, verification is treated as failed.

## Middleware Modes

### Observe Mode (Default)

All requests pass through regardless of verification status. Use this for:

- Logging and analytics
- Gradual rollout
- A/B testing between verified and unverified access

```typescript
app.use(openBotAuthMiddleware({ mode: 'observe' }));
```

### Require-Verified Mode

Protected paths return 401 for unsigned or failed verification:

```typescript
app.use(openBotAuthMiddleware({
  mode: 'require-verified',
  protectedPaths: ['/api/premium', '/api/content']
}));
```

## Error Handling

```typescript
try {
  const result = await client.verify(request);
  if (!result.verified) {
    // Verification failed (invalid signature, expired, etc.)
    console.log('Reason:', result.error);
  }
} catch (error) {
  // Network error, timeout, or verifier service unavailable
  console.error('Verification error:', error);
}
```

## TypeScript Support

The package is written in TypeScript and exports all types:

```typescript
import type {
  VerifierClientOptions,
  VerificationRequest,
  VerificationResult,
  MiddlewareOptions,
  OBAState
} from '@openbotauth/verifier-client';
```

## Examples

### Express with Custom Verifier

```typescript
import express from 'express';
import { openBotAuthMiddleware } from '@openbotauth/verifier-client/express';

const app = express();

app.use(openBotAuthMiddleware({
  verifierUrl: 'https://your-verifier.example.com/verify',
  mode: 'observe',
  timeoutMs: 3000
}));

app.get('/api/data', (req, res) => {
  const { signed, result } = (req as any).oba;

  res.json({
    authenticated: signed && result?.verified,
    agent: result?.agent?.client_name || 'anonymous'
  });
});
```

### Next.js with Body Verification

```typescript
// For POST requests that need body in signature
import { buildVerifyRequestForNextWithBody } from '@openbotauth/verifier-client';

export async function POST(request: NextRequest) {
  const body = await request.text();
  const verifyRequest = buildVerifyRequestForNextWithBody(request, body);
  const result = await client.verify(verifyRequest);
  // ...
}
```

## Links

- **npm:** [https://www.npmjs.com/package/@openbotauth/verifier-client](https://www.npmjs.com/package/@openbotauth/verifier-client)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
