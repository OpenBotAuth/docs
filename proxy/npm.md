# NPM / CLI

Run the OpenBotAuth Proxy using npm or npx.

**Package:** [@openbotauth/proxy](https://www.npmjs.com/package/@openbotauth/proxy)

## Quick Start

```bash
# Run without installing (recommended)
npx @openbotauth/proxy

# Or install globally
npm install -g @openbotauth/proxy
openbotauth-proxy
```

**Requirements:** Node.js >= 18.0.0

## Installation Methods

### npx (No Install)

Run directly without installing:

```bash
npx @openbotauth/proxy
```

### Global Install

```bash
# npm
npm install -g @openbotauth/proxy

# pnpm
pnpm add -g @openbotauth/proxy

# yarn
yarn global add @openbotauth/proxy
```

After global install, run with:

```bash
openbotauth-proxy
# or
oba-proxy
```

### Local Install

Add to your project:

```bash
npm install @openbotauth/proxy
```

Then run via npm scripts in `package.json`:

```json
{
  "scripts": {
    "proxy": "openbotauth-proxy"
  }
}
```

## Configuration

All configuration is via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8088` | Proxy listen port |
| `UPSTREAM_URL` | `http://localhost:8080` | Backend server URL |
| `OBA_VERIFIER_URL` | `https://verifier.openbotauth.org/verify` | Verifier service endpoint |
| `OBA_MODE` | `observe` | `observe` or `require-verified` |
| `OBA_TIMEOUT_MS` | `5000` | Verifier request timeout (ms) |
| `OBA_PROTECTED_PATHS` | (none) | Comma-separated paths requiring verification |

## Usage Examples

### Basic Usage

Proxy requests from port 8088 to localhost:8080:

```bash
npx @openbotauth/proxy
```

### Custom Backend

Proxy to a different backend:

```bash
UPSTREAM_URL=http://localhost:3000 npx @openbotauth/proxy
```

### Custom Port

Run on a different port:

```bash
PORT=9000 npx @openbotauth/proxy
```

### Require Verification

Enforce verification on all paths:

```bash
OBA_MODE=require-verified npx @openbotauth/proxy
```

### Protected Paths Only

Require verification only on specific paths:

```bash
OBA_MODE=require-verified \
OBA_PROTECTED_PATHS=/api,/content,/premium \
npx @openbotauth/proxy
```

### Full Configuration

```bash
PORT=8088 \
UPSTREAM_URL=http://localhost:3000 \
OBA_VERIFIER_URL=https://verifier.openbotauth.org/verify \
OBA_MODE=require-verified \
OBA_TIMEOUT_MS=3000 \
OBA_PROTECTED_PATHS=/api/v1,/protected \
npx @openbotauth/proxy
```

## Process Managers

### PM2

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'oba-proxy',
    script: 'npx',
    args: '@openbotauth/proxy',
    env: {
      PORT: 8088,
      UPSTREAM_URL: 'http://localhost:3000',
      OBA_MODE: 'observe'
    }
  }]
};
```

```bash
pm2 start ecosystem.config.js
```

### systemd

```ini
# /etc/systemd/system/oba-proxy.service
[Unit]
Description=OpenBotAuth Proxy
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/oba-proxy
Environment=PORT=8088
Environment=UPSTREAM_URL=http://localhost:3000
Environment=OBA_MODE=observe
ExecStart=/usr/bin/npx @openbotauth/proxy
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable oba-proxy
sudo systemctl start oba-proxy
```

## Programmatic Usage

You can also use the proxy programmatically in your Node.js application:

```typescript
import { createProxyServer } from '@openbotauth/proxy';

const server = createProxyServer({
  port: 8088,
  upstream: 'http://localhost:3000',
  verifierUrl: 'https://verifier.openbotauth.org/verify',
  mode: 'observe',
  timeoutMs: 5000,
  protectedPaths: ['/api', '/protected']
});

server.listen();
```

## Health Check

Verify the proxy is running:

```bash
curl http://localhost:8088/.well-known/health
```

Response:

```json
{
  "status": "ok",
  "service": "openbotauth-proxy",
  "upstream": "http://localhost:8080",
  "verifier": "https://verifier.openbotauth.org/verify",
  "mode": "observe"
}
```

## Logging

The proxy logs to stdout:

```
[OBA] Proxy started on port 8088
[OBA] Upstream: http://localhost:3000
[OBA] Mode: observe
[OBA] GET /api/content -> verified (agent: MyBot)
[OBA] GET /public -> unsigned
```

## Troubleshooting

### Proxy won't start

Check that the port is available:

```bash
lsof -i :8088
```

### Verification always fails

1. Check the verifier URL is accessible
2. Verify the bot is sending correct RFC 9421 headers
3. Check clock synchronization (signatures expire)

### Timeout errors

Increase the timeout:

```bash
OBA_TIMEOUT_MS=10000 npx @openbotauth/proxy
```

### Backend not receiving requests

1. Verify `UPSTREAM_URL` is correct
2. Check backend is running and accessible
3. Test direct connection to backend

## Links

- **npm:** [https://www.npmjs.com/package/@openbotauth/proxy](https://www.npmjs.com/package/@openbotauth/proxy)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
