# Proxy Overview

The OpenBotAuth Proxy is a reverse proxy that sits in front of your backend and automatically verifies RFC 9421 HTTP message signatures from AI bots.

## Why Use the Proxy?

- **Zero-code integration** - No changes to your application code
- **Language agnostic** - Works with any HTTP backend (Node.js, Python, Go, PHP, etc.)
- **Easy deployment** - Available as npm package, Docker image, or GitHub Container Registry
- **Gradual rollout** - Observe mode lets you monitor before enforcing

## How It Works

```
┌────────────┐      ┌────────────┐      ┌──────────────┐
│   AI Bot   │─────→│   Proxy    │─────→│   Verifier   │
│  (Signed)  │      │  (OBA)     │      │   Service    │
└────────────┘      └────────────┘      └──────────────┘
                           │
                           ↓
                    ┌──────────────┐
                    │   Backend    │
                    │  (Your App)  │
                    └──────────────┘
```

1. AI bot sends a signed HTTP request to the proxy
2. Proxy extracts RFC 9421 signature headers
3. Proxy calls the verifier service to validate the signature
4. Proxy adds `X-OBAuth-*` headers with verification results
5. Proxy forwards the request to your backend
6. Your backend reads the headers to make access decisions

## Installation Options

| Method | Install | Best For |
|--------|---------|----------|
| [NPM / CLI](npm.md) | `npx @openbotauth/proxy` | Quick start, Node.js environments |
| [Docker](docker.md) | `docker pull hammadtariq/openbotauth-proxy` | Containerized deployments |
| [GHCR](ghcr.md) | `docker pull ghcr.io/openbotauth/openbotauth-proxy` | GitHub-integrated workflows |

## Headers Injected

The proxy adds these headers to every request forwarded to your backend:

| Header | Description | Example |
|--------|-------------|---------|
| `X-OBAuth-Verified` | `true` if signature verified, `false` otherwise | `true` |
| `X-OBAuth-Agent` | Bot's client name (if verified) | `MyAIBot` |
| `X-OBAuth-JWKS-URL` | Bot's JWKS URL | `https://registry.openbotauth.org/jwks/bot.json` |
| `X-OBAuth-Kid` | Key ID used for signing | `key-abc123` |
| `X-OBAuth-Error` | Error message (on failure) | `signature_expired` |

## Modes

### Observe Mode (Default)

All requests pass through to backend. Use for:
- Logging and analytics
- Gradual rollout
- Testing before enforcement

```bash
OBA_MODE=observe npx @openbotauth/proxy
```

### Require-Verified Mode

Protected paths return 401 if verification fails:

```bash
OBA_MODE=require-verified \
OBA_PROTECTED_PATHS=/api,/content \
npx @openbotauth/proxy
```

## Configuration

All configuration is via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8088` | Proxy listen port |
| `UPSTREAM_URL` | `http://localhost:8080` | Backend server URL |
| `OBA_VERIFIER_URL` | `https://verifier.openbotauth.org/verify` | Verifier service endpoint |
| `OBA_MODE` | `observe` | `observe` or `require-verified` |
| `OBA_TIMEOUT_MS` | `5000` | Verifier request timeout |
| `OBA_PROTECTED_PATHS` | `/protected` | Comma-separated paths requiring verification |

## Backend Integration

Your backend reads the `X-OBAuth-*` headers to make access decisions:

### Node.js / Express

```javascript
app.get('/api/content', (req, res) => {
  if (req.headers['x-obauth-verified'] === 'true') {
    const agent = req.headers['x-obauth-agent'];
    res.json({ content: 'Full access', bot: agent });
  } else {
    res.json({ content: 'Limited preview' });
  }
});
```

### Python / Flask

```python
@app.route('/api/content')
def content():
    if request.headers.get('X-OBAuth-Verified') == 'true':
        agent = request.headers.get('X-OBAuth-Agent')
        return {'content': 'Full access', 'bot': agent}
    return {'content': 'Limited preview'}
```

### Go

```go
func handler(w http.ResponseWriter, r *http.Request) {
    if r.Header.Get("X-OBAuth-Verified") == "true" {
        agent := r.Header.Get("X-OBAuth-Agent")
        json.NewEncoder(w).Encode(map[string]string{
            "content": "Full access",
            "bot": agent,
        })
    } else {
        json.NewEncoder(w).Encode(map[string]string{
            "content": "Limited preview",
        })
    }
}
```

### PHP

```php
$verified = $_SERVER['HTTP_X_OBAUTH_VERIFIED'] ?? 'false';
$agent = $_SERVER['HTTP_X_OBAUTH_AGENT'] ?? '';

if ($verified === 'true') {
    echo json_encode(['content' => 'Full access', 'bot' => $agent]);
} else {
    echo json_encode(['content' => 'Limited preview']);
}
```

## Health Check

The proxy exposes a health check endpoint:

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

## When to Use Proxy vs SDK

| Use Proxy When | Use SDK When |
|----------------|--------------|
| Backend is not Node.js or Python | Need fine-grained control |
| Want zero-code integration | Integrating with auth system |
| Need language-agnostic solution | Building custom middleware |
| Prefer infrastructure approach | Want to avoid extra hop |

## Links

- **npm:** [https://www.npmjs.com/package/@openbotauth/proxy](https://www.npmjs.com/package/@openbotauth/proxy)
- **Docker Hub:** [https://hub.docker.com/r/hammadtariq/openbotauth-proxy](https://hub.docker.com/r/hammadtariq/openbotauth-proxy)
- **GitHub Container Registry:** [https://github.com/OpenBotAuth/openbotauth/pkgs/container/openbotauth-proxy](https://github.com/OpenBotAuth/openbotauth/pkgs/container/openbotauth-proxy)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
