# @openbotauth/registry-signer

A TypeScript/JavaScript library for generating Ed25519 key pairs and formatting them as JSON Web Keys (JWK) for use with OpenBotRegistry.

## Installation

```bash
npm install @openbotauth/registry-signer
```

## Quick Start

```typescript
import {
  generateEd25519KeyPair,
  publicKeyToJwk,
  privateKeyToJwk,
  createJwks
} from '@openbotauth/registry-signer';

// Generate a new Ed25519 key pair
const keyPair = await generateEd25519KeyPair();

// Export public key as JWK (for registration)
const publicJwk = await publicKeyToJwk(keyPair.publicKey, 'my-key-id');

// Export private key as JWK (for signing - keep secure!)
const privateJwk = await privateKeyToJwk(keyPair.privateKey, 'my-key-id');

// Create a JWKS containing your public key
const jwks = await createJwks([publicJwk]);
console.log(JSON.stringify(jwks, null, 2));
```

## API Reference

### Key Generation

#### `generateEd25519KeyPair()`

Generates a new Ed25519 key pair using the Web Crypto API.

```typescript
const keyPair = await generateEd25519KeyPair();
// Returns: CryptoKeyPair { publicKey, privateKey }
```

### JWK Conversion

#### `publicKeyToJwk(publicKey, keyId?)`

Converts a CryptoKey public key to JWK format.

```typescript
const jwk = await publicKeyToJwk(keyPair.publicKey, 'optional-key-id');
// Returns:
// {
//   kty: 'OKP',
//   crv: 'Ed25519',
//   x: 'base64url-encoded-public-key',
//   kid: 'optional-key-id'
// }
```

#### `privateKeyToJwk(privateKey, keyId?)`

Converts a CryptoKey private key to JWK format.

```typescript
const jwk = await privateKeyToJwk(keyPair.privateKey, 'optional-key-id');
// Returns:
// {
//   kty: 'OKP',
//   crv: 'Ed25519',
//   x: 'base64url-encoded-public-key',
//   d: 'base64url-encoded-private-key',
//   kid: 'optional-key-id'
// }
```

### JWKS Creation

#### `createJwks(keys)`

Creates a JSON Web Key Set from an array of JWKs.

```typescript
const jwks = await createJwks([publicJwk]);
// Returns:
// {
//   keys: [
//     { kty: 'OKP', crv: 'Ed25519', x: '...', kid: '...' }
//   ]
// }
```

### Base64 Utilities

#### `base64UrlEncode(data)`

Encodes a Uint8Array to base64url format (no padding).

```typescript
const encoded = base64UrlEncode(new Uint8Array([1, 2, 3]));
```

#### `base64UrlDecode(str)`

Decodes a base64url string to Uint8Array.

```typescript
const decoded = base64UrlDecode('AQID');
```

## Hosting Your JWKS

Once you've generated your key pair, host the JWKS at a publicly accessible URL:

### Option 1: Static File

Save the JWKS output to a file and serve it:

```bash
# Generate and save JWKS
node -e "
const { generateEd25519KeyPair, publicKeyToJwk, createJwks } = require('@openbotauth/registry-signer');
(async () => {
  const keyPair = await generateEd25519KeyPair();
  const jwk = await publicKeyToJwk(keyPair.publicKey, 'key-1');
  const jwks = await createJwks([jwk]);
  console.log(JSON.stringify(jwks, null, 2));
})();
" > .well-known/jwks.json
```

### Option 2: Dynamic Endpoint

Serve the JWKS from your application:

```typescript
// Express.js example
app.get('/.well-known/jwks.json', (req, res) => {
  res.json({
    keys: [
      {
        kty: 'OKP',
        crv: 'Ed25519',
        x: process.env.PUBLIC_KEY_X,
        kid: 'crawler-key-1'
      }
    ]
  });
});
```

## Security Considerations

- **Never expose your private key** - The private key (`d` parameter) should never be shared or included in your JWKS
- **Store private keys securely** - Use environment variables, secret managers, or hardware security modules
- **Use unique key IDs** - Include a `kid` to identify keys during rotation
- **Rotate keys periodically** - Generate new keys and update your registration

## TypeScript Support

Full TypeScript definitions are included. The package exports proper types for all functions and return values.

## Browser Compatibility

This package uses the Web Crypto API and works in:

- Node.js 18+
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Deno
- Cloudflare Workers

## Source Code

GitHub: [OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth/tree/main/packages/registry-signer)

npm: [@openbotauth/registry-signer](https://www.npmjs.com/package/@openbotauth/registry-signer)
