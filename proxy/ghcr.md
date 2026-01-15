# GitHub Container Registry

Run the OpenBotAuth Proxy using GitHub Container Registry (GHCR) images.

**Image:** [ghcr.io/openbotauth/openbotauth-proxy](https://github.com/OpenBotAuth/openbotauth/pkgs/container/openbotauth-proxy)

## Why GHCR?

- **GitHub integration** - Works seamlessly with GitHub Actions and workflows
- **Same source** - Images built directly from GitHub repository
- **Alternative registry** - Fallback if Docker Hub is unavailable

## Quick Start

```bash
docker run -p 8088:8088 ghcr.io/openbotauth/openbotauth-proxy
```

## Supported Platforms

- `linux/amd64` - Intel/AMD 64-bit
- `linux/arm64` - ARM 64-bit (Apple Silicon, AWS Graviton)

## Pull Image

```bash
# Latest version
docker pull ghcr.io/openbotauth/openbotauth-proxy

# Specific version
docker pull ghcr.io/openbotauth/openbotauth-proxy:0.1.5

# Latest tag
docker pull ghcr.io/openbotauth/openbotauth-proxy:latest
```

## Authentication

GHCR images are public, but for private images or higher rate limits:

```bash
# Login with GitHub token
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

## Configuration

Same environment variables as Docker Hub image:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8088` | Proxy listen port |
| `UPSTREAM_URL` | `http://localhost:8080` | Backend server URL |
| `OBA_VERIFIER_URL` | `https://verifier.openbotauth.org/verify` | Verifier endpoint |
| `OBA_MODE` | `observe` | `observe` or `require-verified` |
| `OBA_TIMEOUT_MS` | `5000` | Verifier timeout (ms) |
| `OBA_PROTECTED_PATHS` | (none) | Comma-separated protected paths |

## Usage Examples

### Basic Usage

```bash
docker run -p 8088:8088 ghcr.io/openbotauth/openbotauth-proxy
```

### With Configuration

```bash
docker run -p 8088:8088 \
  -e UPSTREAM_URL=http://backend:3000 \
  -e OBA_MODE=require-verified \
  -e OBA_PROTECTED_PATHS=/api,/content \
  ghcr.io/openbotauth/openbotauth-proxy
```

## Docker Compose

```yaml
version: '3.8'

services:
  proxy:
    image: ghcr.io/openbotauth/openbotauth-proxy
    ports:
      - "8088:8088"
    environment:
      - UPSTREAM_URL=http://backend:3000
      - OBA_MODE=observe
    depends_on:
      - backend

  backend:
    image: your-backend-image
    expose:
      - "3000"
```

## GitHub Actions

### Use in Workflows

```yaml
name: Deploy with OBA Proxy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    services:
      oba-proxy:
        image: ghcr.io/openbotauth/openbotauth-proxy
        ports:
          - 8088:8088
        env:
          UPSTREAM_URL: http://localhost:3000
          OBA_MODE: observe

    steps:
      - uses: actions/checkout@v4

      - name: Start backend
        run: npm start &

      - name: Test through proxy
        run: curl http://localhost:8088/.well-known/health
```

### Build and Deploy with GHCR

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull OBA Proxy
        run: docker pull ghcr.io/openbotauth/openbotauth-proxy

      - name: Deploy
        run: |
          docker run -d \
            -p 8088:8088 \
            -e UPSTREAM_URL=http://backend:3000 \
            ghcr.io/openbotauth/openbotauth-proxy
```

## Kubernetes with GHCR

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oba-proxy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: oba-proxy
  template:
    metadata:
      labels:
        app: oba-proxy
    spec:
      containers:
        - name: oba-proxy
          image: ghcr.io/openbotauth/openbotauth-proxy
          ports:
            - containerPort: 8088
          env:
            - name: UPSTREAM_URL
              value: "http://backend-service:3000"
            - name: OBA_MODE
              value: "observe"
```

For private images, add imagePullSecrets:

```yaml
spec:
  imagePullSecrets:
    - name: ghcr-secret
  containers:
    - name: oba-proxy
      image: ghcr.io/openbotauth/openbotauth-proxy
```

## Comparing Registries

| Feature | Docker Hub | GHCR |
|---------|------------|------|
| URL | `hammadtariq/openbotauth-proxy` | `ghcr.io/openbotauth/openbotauth-proxy` |
| Auth | Docker Hub account | GitHub token |
| Rate Limits | 100 pulls/6hr (anon) | 1000+ pulls/hr |
| GitHub Actions | Manual login | Native integration |
| Same Image | Yes | Yes |

## Health Check

```bash
curl http://localhost:8088/.well-known/health
```

Response:

```json
{
  "status": "ok",
  "service": "openbotauth-proxy",
  "upstream": "http://backend:3000",
  "verifier": "https://verifier.openbotauth.org/verify",
  "mode": "observe"
}
```

## Troubleshooting

### Rate limit exceeded

Login to GHCR for higher limits:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

### Image not found

Verify the image exists:

```bash
docker manifest inspect ghcr.io/openbotauth/openbotauth-proxy
```

### Permission denied

For private images, ensure your token has `read:packages` scope.

## Links

- **GHCR:** [https://github.com/OpenBotAuth/openbotauth/pkgs/container/openbotauth-proxy](https://github.com/OpenBotAuth/openbotauth/pkgs/container/openbotauth-proxy)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
