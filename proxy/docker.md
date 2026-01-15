# Docker

Run the OpenBotAuth Proxy using Docker Hub images.

**Image:** [hammadtariq/openbotauth-proxy](https://hub.docker.com/r/hammadtariq/openbotauth-proxy)

## Quick Start

```bash
docker run -p 8088:8088 hammadtariq/openbotauth-proxy
```

## Supported Platforms

The Docker image supports multiple architectures:

- `linux/amd64` - Intel/AMD 64-bit
- `linux/arm64` - ARM 64-bit (Apple Silicon, AWS Graviton)

## Pull Image

```bash
# Latest version
docker pull hammadtariq/openbotauth-proxy

# Specific version
docker pull hammadtariq/openbotauth-proxy:0.1.5

# Latest tag
docker pull hammadtariq/openbotauth-proxy:latest
```

## Configuration

Configure via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8088` | Proxy listen port |
| `UPSTREAM_URL` | `http://localhost:8080` | Backend server URL |
| `OBA_VERIFIER_URL` | `https://verifier.openbotauth.org/verify` | Verifier endpoint |
| `OBA_MODE` | `observe` | `observe` or `require-verified` |
| `OBA_TIMEOUT_MS` | `5000` | Verifier timeout (ms) |
| `OBA_PROTECTED_PATHS` | `/protected` | Comma-separated protected paths |

## Usage Examples

### Basic Usage

```bash
docker run -p 8088:8088 hammadtariq/openbotauth-proxy
```

### Custom Backend

```bash
docker run -p 8088:8088 \
  -e UPSTREAM_URL=http://host.docker.internal:3000 \
  hammadtariq/openbotauth-proxy
```

### Require Verification

```bash
docker run -p 8088:8088 \
  -e UPSTREAM_URL=http://backend:3000 \
  -e OBA_MODE=require-verified \
  -e OBA_PROTECTED_PATHS=/api,/content \
  hammadtariq/openbotauth-proxy
```

### Full Configuration

```bash
docker run -p 8088:8088 \
  -e PORT=8088 \
  -e UPSTREAM_URL=http://backend:3000 \
  -e OBA_VERIFIER_URL=https://verifier.openbotauth.org/verify \
  -e OBA_MODE=require-verified \
  -e OBA_TIMEOUT_MS=3000 \
  -e OBA_PROTECTED_PATHS=/api/v1,/protected \
  hammadtariq/openbotauth-proxy
```

## Docker Compose

### Basic Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  proxy:
    image: hammadtariq/openbotauth-proxy
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

### Production Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  proxy:
    image: hammadtariq/openbotauth-proxy
    ports:
      - "8088:8088"
    environment:
      - UPSTREAM_URL=http://backend:3000
      - OBA_MODE=require-verified
      - OBA_PROTECTED_PATHS=/api,/content
      - OBA_TIMEOUT_MS=3000
    depends_on:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8088/.well-known/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  backend:
    image: your-backend-image
    expose:
      - "3000"
    restart: unless-stopped
```

### With Nginx Frontend

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - proxy

  proxy:
    image: hammadtariq/openbotauth-proxy
    expose:
      - "8088"
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

## Kubernetes

### Deployment

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
          image: hammadtariq/openbotauth-proxy
          ports:
            - containerPort: 8088
          env:
            - name: UPSTREAM_URL
              value: "http://backend-service:3000"
            - name: OBA_MODE
              value: "observe"
          livenessProbe:
            httpGet:
              path: /.well-known/health
              port: 8088
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /.well-known/health
              port: 8088
            initialDelaySeconds: 5
            periodSeconds: 5
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: oba-proxy
spec:
  selector:
    app: oba-proxy
  ports:
    - port: 8088
      targetPort: 8088
  type: ClusterIP
```

## Networking

### Accessing Host Services

When proxying to services on the Docker host:

**Docker Desktop (Mac/Windows):**
```bash
docker run -p 8088:8088 \
  -e UPSTREAM_URL=http://host.docker.internal:3000 \
  hammadtariq/openbotauth-proxy
```

**Linux:**
```bash
docker run -p 8088:8088 \
  --add-host=host.docker.internal:host-gateway \
  -e UPSTREAM_URL=http://host.docker.internal:3000 \
  hammadtariq/openbotauth-proxy
```

### Docker Network

For containers in the same network:

```bash
# Create network
docker network create oba-network

# Run backend
docker run -d --name backend --network oba-network your-backend

# Run proxy
docker run -p 8088:8088 --network oba-network \
  -e UPSTREAM_URL=http://backend:3000 \
  hammadtariq/openbotauth-proxy
```

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

### Cannot connect to backend

1. Ensure backend is on the same Docker network
2. Use service name (not localhost) for `UPSTREAM_URL`
3. Check backend is exposing the correct port

### Image pull fails

```bash
# Check Docker Hub status
docker pull hammadtariq/openbotauth-proxy

# Try with explicit registry
docker pull docker.io/hammadtariq/openbotauth-proxy
```

### Container exits immediately

Check container logs:

```bash
docker logs <container_id>
```

## Links

- **Docker Hub:** [https://hub.docker.com/r/hammadtariq/openbotauth-proxy](https://hub.docker.com/r/hammadtariq/openbotauth-proxy)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
