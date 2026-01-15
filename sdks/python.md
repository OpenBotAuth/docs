# Python SDK

The official Python SDK for OpenBotAuth signature verification.

**Package:** [openbotauth-verifier](https://pypi.org/project/openbotauth-verifier/)

## Installation

```bash
# Core package
pip install openbotauth-verifier

# With FastAPI/Starlette support
pip install openbotauth-verifier[fastapi]

# With Flask support
pip install openbotauth-verifier[flask]

# All extras
pip install openbotauth-verifier[all]
```

**Requirements:** Python >= 3.10

## Quick Start

### FastAPI Middleware

The fastest way to integrate with FastAPI:

```python
from fastapi import FastAPI, Request
from openbotauth_verifier import OpenBotAuthASGIMiddleware

app = FastAPI()

# Add middleware (observe mode by default)
app.add_middleware(OpenBotAuthASGIMiddleware)

@app.get("/api/content")
async def get_content(request: Request):
    oba = request.state.oba

    if oba.signed and oba.result.verified:
        # Verified bot - full access
        return {
            "content": "Full article content...",
            "agent": oba.result.agent
        }
    else:
        # Anonymous or unverified - limited access
        return {
            "content": "Article preview...",
            "upgrade": "Sign requests for full access"
        }
```

### Flask Middleware

For Flask applications:

```python
from flask import Flask, request, g
from openbotauth_verifier.middleware.wsgi import OpenBotAuthWSGIMiddleware

app = Flask(__name__)

# Wrap WSGI app
app.wsgi_app = OpenBotAuthWSGIMiddleware(app.wsgi_app)

@app.before_request
def load_oba():
    """Load OBA state from WSGI environ into Flask's g object"""
    g.oba = request.environ.get("openbotauth.oba")

@app.route("/api/content")
def get_content():
    if g.oba and g.oba.signed and g.oba.result.verified:
        return {
            "content": "Full article content...",
            "agent": g.oba.result.agent
        }
    return {
        "content": "Article preview...",
        "upgrade": "Sign requests for full access"
    }
```

### Direct Client Usage

For custom integrations:

```python
from openbotauth_verifier import VerifierClient

client = VerifierClient(
    verifier_url="https://verifier.openbotauth.org/verify",  # default
    timeout_s=5.0  # default
)

# Async usage
result = await client.verify(
    method="GET",
    url="https://example.com/api/content",
    headers={
        "host": "example.com",
        "signature-input": 'sig=("@method" "@target-uri" "host");created=1699900000;keyid="key-1";alg="ed25519"',
        "signature": "sig=:base64signature...:",
        "signature-agent": "https://registry.openbotauth.org/jwks/mybot.json"
    }
)

if result.verified:
    print(f"Verified agent: {result.agent['client_name']}")
else:
    print(f"Verification failed: {result.error}")

# Sync usage
result = client.verify_sync(
    method="GET",
    url="https://example.com/api/content",
    headers={...}
)
```

## API Reference

### VerifierClient

Main client class for calling the verifier service.

```python
class VerifierClient:
    def __init__(
        self,
        verifier_url: str = "https://verifier.openbotauth.org/verify",
        timeout_s: float = 5.0,
    ):
        ...

    async def verify(
        self,
        method: str,
        url: str,
        headers: dict[str, str],
        body: str | None = None
    ) -> VerificationResult:
        """Async verification"""
        ...

    def verify_sync(
        self,
        method: str,
        url: str,
        headers: dict[str, str],
        body: str | None = None
    ) -> VerificationResult:
        """Synchronous verification"""
        ...
```

### VerificationRequest

Request data model:

```python
from dataclasses import dataclass

@dataclass
class VerificationRequest:
    method: str              # HTTP method (GET, POST, etc.)
    url: str                 # Full request URL
    headers: dict[str, str]  # Request headers
    body: str | None = None  # Request body (for POST/PUT)
```

### VerificationResult

Response from the verifier:

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class VerificationResult:
    verified: bool                      # Whether signature is valid
    agent: dict[str, Any] | None = None # Agent info (if verified)
    error: str | None = None            # Error message (if failed)
    created: int | None = None          # Signature creation timestamp
    expires: int | None = None          # Signature expiration timestamp
```

### OBAState

State attached to requests by middleware:

```python
@dataclass
class OBAState:
    signed: bool                         # Request had signature headers
    result: VerificationResult | None = None  # Verification result
```

## Middleware Configuration

### ASGI Middleware (FastAPI/Starlette)

```python
from openbotauth_verifier import OpenBotAuthASGIMiddleware

app.add_middleware(
    OpenBotAuthASGIMiddleware,
    verifier_url="https://verifier.openbotauth.org/verify",  # optional
    require_verified=False,  # True to enforce verification
    timeout_s=5.0    # optional
)
```

### WSGI Middleware (Flask)

```python
from openbotauth_verifier.middleware.wsgi import OpenBotAuthWSGIMiddleware

app.wsgi_app = OpenBotAuthWSGIMiddleware(
    app.wsgi_app,
    verifier_url="https://verifier.openbotauth.org/verify",  # optional
    require_verified=False,  # True to enforce verification
    timeout_s=5.0    # optional
)
```

## Header Utilities

Utility functions for working with RFC 9421 headers:

```python
from openbotauth_verifier import (
    has_signature_headers,
    parse_covered_headers,
    extract_forwarded_headers
)

# Check if request has signature headers
has_sig = has_signature_headers(headers)

# Parse covered headers from Signature-Input
covered = parse_covered_headers(signature_input)
# Returns: ['@method', '@target-uri', 'host', ...]

# Extract only safe headers for forwarding to verifier
safe_headers = extract_forwarded_headers(headers, covered)
```

## Security

### Sensitive Headers

The SDK automatically blocks sensitive headers from being forwarded to the verifier:

- `cookie`
- `authorization`
- `proxy-authorization`
- `www-authenticate`

If a `Signature-Input` references any of these headers, a `ValueError` is raised.

### Timeout Handling

All verification requests have a configurable timeout (default 5 seconds). On timeout, verification is treated as failed.

## Middleware Modes

### Observe Mode (Default)

All requests pass through regardless of verification status:

```python
app.add_middleware(OpenBotAuthASGIMiddleware, require_verified=False)
```

### Require-Verified Mode

Returns 401 for unsigned or failed verification:

```python
app.add_middleware(OpenBotAuthASGIMiddleware, require_verified=True)
```

## Error Handling

```python
from openbotauth_verifier import VerifierClient
from httpx import HTTPError

client = VerifierClient()

try:
    result = await client.verify(method="GET", url="...", headers={...})
    if not result.verified:
        # Verification failed (invalid signature, expired, etc.)
        print(f"Reason: {result.error}")
except HTTPError as e:
    # Network error, timeout, or verifier service unavailable
    print(f"Verification error: {e}")
```

## Type Hints

The package includes full type hints and is compatible with mypy:

```python
from openbotauth_verifier import (
    VerifierClient,
    VerificationRequest,
    VerificationResult,
    OBAState
)
```

## Examples

### FastAPI with Custom Verifier

```python
from fastapi import FastAPI, Request, HTTPException
from openbotauth_verifier import OpenBotAuthASGIMiddleware

app = FastAPI()

app.add_middleware(
    OpenBotAuthASGIMiddleware,
    verifier_url="https://your-verifier.example.com/verify",
    require_verified=False,
    timeout_s=3.0
)

@app.get("/api/data")
async def get_data(request: Request):
    oba = request.state.oba

    return {
        "authenticated": oba.signed and oba.result.verified if oba.result else False,
        "agent": oba.result.agent.get("client_name") if oba.result and oba.result.agent else "anonymous"
    }
```

### Starlette Direct Usage

```python
from starlette.applications import Starlette
from starlette.routing import Route
from starlette.responses import JSONResponse
from openbotauth_verifier import OpenBotAuthASGIMiddleware

async def homepage(request):
    oba = request.state.oba
    if oba.signed and oba.result.verified:
        return JSONResponse({"message": "Hello verified bot!"})
    return JSONResponse({"message": "Hello anonymous!"})

app = Starlette(routes=[Route("/", homepage)])
app = OpenBotAuthASGIMiddleware(app)
```

### Flask with Protected Routes

```python
from functools import wraps
from flask import Flask, request, g, jsonify
from openbotauth_verifier.middleware.wsgi import OpenBotAuthWSGIMiddleware

app = Flask(__name__)
app.wsgi_app = OpenBotAuthWSGIMiddleware(app.wsgi_app)

@app.before_request
def load_oba():
    g.oba = request.environ.get("openbotauth.oba")

def require_verified(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if not g.oba or not g.oba.signed or not g.oba.result.verified:
            return jsonify({"error": "Verification required"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/api/public")
def public():
    return jsonify({"message": "Public content"})

@app.route("/api/protected")
@require_verified
def protected():
    return jsonify({
        "message": "Protected content",
        "agent": g.oba.result.agent
    })
```

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| httpx | >= 0.25.0 | HTTP client (async & sync) |
| fastapi | >= 0.100 | FastAPI framework (optional) |
| starlette | >= 0.27 | ASGI framework (optional) |
| flask | >= 2.0 | Flask framework (optional) |

## Links

- **PyPI:** [https://pypi.org/project/openbotauth-verifier/](https://pypi.org/project/openbotauth-verifier/)
- **GitHub:** [https://github.com/OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth)
