# WordPress Plugin

The OpenBotAuth WordPress plugin enables content owners to control how AI agents and bots access their content using RFC 9421 HTTP Message Signatures.

## Features

- **Signature Verification** - Verify bot identity using Ed25519 cryptographic signatures
- **Content Teasers** - Show first N words to unverified bots
- **Payment Flow** - Return 402 Payment Required for premium content
- **Rate Limiting** - Per-agent rate limits to prevent abuse
- **Access Control** - Whitelist/blacklist specific bots
- **Per-Post Policies** - Override default policy on individual posts

## Requirements

- WordPress 6.0 or higher
- PHP 7.4 or higher
- Access to OpenBotAuth Verifier Service

## Installation

### Install from WordPress.org (Recommended)

1. Go to **WordPress Admin → Plugins → Add New**
2. Search for **"OpenBotAuth"**
3. Click **Install Now**
4. Click **Activate**

Or install directly: [wordpress.org/plugins/openbotauth](https://wordpress.org/plugins/openbotauth/)

### Manual Installation (Development)

For development or testing the latest changes:

```bash
# Clone the repository
git clone https://github.com/OpenBotAuth/openbotauth.git

# Copy plugin to WordPress
cp -r openbotauth/plugins/wordpress-openbotauth /path/to/wordpress/wp-content/plugins/
```

Then activate via **WordPress Admin → Plugins**.

## Configuration

### Basic Setup

1. Go to **Settings → OpenBotAuth**

2. Configure the **Verifier Service URL**:

   | Environment | URL |
   |-------------|-----|
   | Production (hosted) | `https://verifier.openbotauth.org/verify` |
   | Self-hosted | `https://verifier.yourdomain.com/verify` |
   | Local development | `http://localhost:8081/verify` |

3. Set **Default Policy**:
   - **Allow** - All bots can access content
   - **Teaser** - Show preview to unverified bots (recommended)
   - **Deny** - Block unverified bots

4. Set **Teaser Word Count** (default: 100)

5. Click **Save Settings**

### Per-Post Policies

Override the default policy for individual posts:

1. Edit a post or page
2. Find the **OpenBotAuth Policy** meta box in the sidebar
3. Check **Override default policy**
4. Configure:
   - **Effect**: Allow, Teaser, or Deny
   - **Teaser Words**: Number of words for preview
   - **Price (cents)**: Require payment (e.g., `500` for $5.00)
5. Save the post

### Advanced Policy Configuration

For advanced policies, edit the Policy JSON directly in settings:

```json
{
  "default": {
    "effect": "teaser",
    "teaser_words": 100,
    "whitelist": [
      "https://trusted-bot.example.com/jwks.json"
    ],
    "blacklist": [
      "https://badbot.example.com/*"
    ],
    "rate_limit": {
      "max_requests": 100,
      "window_seconds": 3600
    }
  }
}
```

## Policy Options

| Field | Type | Description |
|-------|------|-------------|
| `effect` | string | Default action: `allow`, `deny`, or `teaser` |
| `teaser_words` | number | Words to show in preview (0 = no teaser) |
| `price_cents` | number | Price in cents (0 = free, >0 = 402 response) |
| `currency` | string | Currency code (default: `USD`) |
| `whitelist` | array | Bot patterns to always allow |
| `blacklist` | array | Bot patterns to always deny |
| `rate_limit.max_requests` | number | Max requests per window |
| `rate_limit.window_seconds` | number | Time window in seconds |

## Response Headers

The plugin adds an `X-OBA-Decision` header to responses:

| Value | Meaning |
|-------|---------|
| `allow` | Bot is verified and allowed full access |
| `teaser` | Unverified bot receives preview content |
| `pay` | Payment required (402 response) |
| `deny` | Bot is denied access (403 response) |
| `rate_limit` | Rate limit exceeded (429 response) |

## Hooks and Filters

### Filter: `openbotauth_policy`

Modify policy before applying:

```php
add_filter('openbotauth_policy', function($policy, $post) {
    if ($post->post_type === 'premium') {
        $policy['price_cents'] = 1000;
    }
    return $policy;
}, 10, 2);
```

### Action: `openbotauth_verified`

Triggered when a bot is verified:

```php
add_action('openbotauth_verified', function($agent, $post) {
    error_log("Bot {$agent['jwks_url']} accessed post {$post->ID}");
}, 10, 2);
```

### Action: `openbotauth_payment_required`

Triggered when 402 is returned:

```php
add_action('openbotauth_payment_required', function($agent, $post, $price) {
    // Track payment requests
}, 10, 3);
```

## Troubleshooting

### Verifier Connection Failed

**Error**: "Verifier service error: Connection refused"

1. Check verifier service is running
2. Verify URL in Settings → OpenBotAuth
3. Check firewall rules

### Teaser Not Showing

1. Verify policy effect is set to `teaser`
2. Ensure `teaser_words` > 0
3. Log out of WordPress (logged-in users see full content)
4. Check `X-OBA-Decision` header in response

### No X-OBA-Decision Header

1. Ensure you're testing on a singular post/page (not homepage)
2. Log out of WordPress
3. Check PHP error logs for verifier connection issues

## Links

- **WordPress.org:** [wordpress.org/plugins/openbotauth](https://wordpress.org/plugins/openbotauth/)
- **GitHub:** [OpenBotAuth/openbotauth](https://github.com/OpenBotAuth/openbotauth/tree/main/plugins/wordpress-openbotauth)

## License

GPLv2 or later
