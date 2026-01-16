# Crawlers Overview

Crawlers (AI agents) register with OpenBotRegistry to establish their cryptographic identity. This identity enables them to sign HTTP requests using RFC 9421 HTTP Message Signatures, which publishers can then verify.

## Why Register?

When your crawler is registered with OpenBotRegistry:

- **Publishers can verify your identity** - Your signed requests prove you are who you claim to be
- **Access to premium content** - Publishers can grant access to registered, verified crawlers
- **Pay-per-crawl support** - Enable monetized content access through the Web Bot Auth payment protocol
- **Trust building** - Build reputation as a legitimate AI agent

## Registration Process

### 1. Generate Ed25519 Key Pair

Use the [@openbotauth/registry-signer](registry-signer.md) package to generate an Ed25519 key pair for signing requests.

```bash
npm install @openbotauth/registry-signer
```

```typescript
import { generateEd25519KeyPair, publicKeyToJwk } from '@openbotauth/registry-signer';

const keyPair = await generateEd25519KeyPair();
const jwk = await publicKeyToJwk(keyPair.publicKey);

console.log('JWK:', JSON.stringify(jwk, null, 2));
```

### 2. Create JWKS Endpoint

Host your public key as a JSON Web Key Set (JWKS) at a well-known URL:

```json
{
  "keys": [
    {
      "kty": "OKP",
      "crv": "Ed25519",
      "x": "your-base64url-encoded-public-key",
      "kid": "your-key-id"
    }
  ]
}
```

Example URL: `https://your-domain.com/.well-known/jwks.json`

### 3. Register with OpenBotRegistry

Submit your crawler registration at [OpenBotRegistry Portal](https://registry.openbotauth.org):

| Field | Description |
|-------|-------------|
| **Crawler Name** | Human-readable name for your AI agent |
| **Organization** | Your company or project name |
| **JWKS URL** | URL where your public key is hosted |
| **Contact Email** | For verification and communication |

### 4. Sign Your Requests

Use your private key to sign HTTP requests according to RFC 9421:

```typescript
// Pseudo-code for request signing
const signature = sign(privateKey, {
  method: 'GET',
  url: 'https://publisher.example.com/content',
  headers: {
    'host': 'publisher.example.com',
    'date': new Date().toISOString()
  }
});

// Add signature headers to your request
headers['Signature-Input'] = '...';
headers['Signature'] = '...';
```

## Key Management Best Practices

- **Keep private keys secure** - Never expose your private key in client-side code or public repositories
- **Rotate keys regularly** - Update your JWKS and re-register with a new key periodically
- **Use key IDs** - Include a unique `kid` in your JWK for key identification
- **Monitor usage** - Track which requests are being made with your identity

## Verification Flow

When a publisher receives your signed request:

```
1. Extract signature headers from request
2. Fetch your JWKS from the registered URL
3. Verify the signature using your public key
4. Check that the key belongs to a registered crawler
5. Make policy decision (allow, deny, or require payment)
```

## Related Resources

- [registry-signer Package](registry-signer.md) - Generate Ed25519 keys for signing
- [Architecture Overview](../architecture/overview.md) - How OpenBotAuth works
- [RFC 9421](https://datatracker.ietf.org/doc/html/rfc9421) - HTTP Message Signatures specification
