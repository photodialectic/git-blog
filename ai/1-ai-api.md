# AI-API: Self-Hosted Bifrost Gateway

Managing multiple AI provider APIs (OpenAI, Anthropic, Google) across different services becomes unwieldy quickly. I prefer a unified gateway approach. I started with the LiteLLM reverse proxy and later switched to Bifrost for two main reasons:

1. Smaller memory footprint: LiteLLM's reverse proxy consumed roughly 500-1000 MB in my setup, which is significant for a single low-cost host.
2. Less configuration overhead: I am not using advanced governance/rate-management features, and Bifrost's pass-through model configuration reduced alias and routing management for my use case.

## Why Bifrost?

[Bifrost](https://docs.getbifrost.ai/overview) translates requests between different AI provider APIs, presenting a unified OpenAI-compatible interface. This means:

- **Single integration point** - all services use the same API format
- **Provider abstraction** - switch between models without changing application code
- **Centralized authentication** - one master key instead of multiple provider keys
- **Built-in features** - logging, rate limiting, cost tracking, and fallbacks

## Configuration

My AI-API runs as a Docker container in the HomeStack with this Bifrost configuration:

```javascript
{
  "$schema": "https://www.getbifrost.ai/schema",
  "providers": {
    "openai": {
      "keys": [
        { "name": "openai-primary", "value": "env.OPENAI_API_KEY", "weight": 1 },
      ]
    },
    "anthropic": {
      "keys": [
        { "name": "anthropic-primary", "value": "env.ANTHROPIC_API_KEY", "weight": 1 }
      ]
    },
    "gemini": {
      "keys": [
        { "name": "gemini-primary", "value": "env.GEMINI_API_KEY", "weight": 1 }
      ]
    },
    "vertex": {
      "keys": [
        {
          "name": "vertex-primary",
          "weight": 1,
          "vertex_key_config": {
            "project_id": "vertex-487414",
            "region": "global",
            "auth_credentials": "env.VERTEX_CREDENTIALS_JSON"
          }
        }
      ]
    }
  },
  "governance": {
    "virtual_keys": [
      {
        "id": "vk-master",
        "name": "master",
        "value": "env.AI_API_MK",
        "is_active": true,
        "provider_configs": [
          { "provider": "openai" },
          { "provider": "anthropic" },
          { "provider": "gemini" },
          { "provider": "vertex" }
        ]
      }
    ]
  },
  "auth_config": {
    "admin_username": "env.AI_API_ADMIN_USER",
    "admin_password": "env.AI_API_ADMIN_PASS",
    "is_enabled": true,
    "disable_auth_on_inference": true
  },
  "client": {
    "enforce_governance_header": true,
    "enforce_auth_on_inference": true,
    "enable_litellm_fallback": true
  },
  "config_store": {
    "enabled": true,
    "type": "sqlite",
    "config": {
      "path": "/app/db/bifrost.db"
    }
  }
}
```

## Docker Integration

The AI-API service integrates seamlessly into my Docker Compose stack:

```yaml
ai-api:
    container_name: ai-api
    environment:
        - OPENAI_API_KEY=${OPENAI_API_KEY}
        - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
        - OPENAI_API_KEY_BF=${OPENAI_API_KEY_BF}
        - ANTHROPIC_API_KEY_BF=${ANTHROPIC_API_KEY_BF}
        - AI_API_MK=${AI_API_MK}
        - GEMINI_API_KEY=${GEMINI_API_KEY}
        - GOOGLE_APPLICATION_CREDENTIALS=${VERTEX_CREDENTIALS_JSON}
        - VERTEX_CREDENTIALS_JSON=${VERTEX_CREDENTIALS_JSON}
        - AI_API_ADMIN_USER=${AI_API_ADMIN_USER}
        - AI_API_ADMIN_PASS=${AI_API_ADMIN_PASS}
        - APP_PORT=10200
    image: maximhq/bifrost:latest
    labels:
        - traefik.enable=true
        - traefik.http.services.ai-api.loadbalancer.server.port=10200
        - traefik.http.routers.ai-api.middlewares=ai-api-stripprefix
        - traefik.http.middlewares.ai-api-stripprefix.stripprefix.prefixes=/ai-api
    ports:
        - 10200:10200
    profiles:
        - site
        - dev
        - prod
        - stage
    restart: unless-stopped
    volumes:
        - ./ai-api/config.json:/app/data/config.json:ro
        - ./ai-api/db:/app/db
```

Key configuration details:

- **Path-based routing** - accessible at `www.nickhedberg.com/ai-api/v1`
- **Environment variables** - provider API keys loaded from encrypted secrets
- **Volume mount** - configuration file mounted into container
- **Master key authentication** - single key controls access to all models

## Usage Examples

Services can now use any AI model through a single endpoint:

### Chat Completion

```javascript
const response = await fetch(
  "https://www.nickhedberg.com/ai-api/v1/chat/completions",
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${AI_API_MK}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "anthropic/claude-4-5-sonnet",
      messages: [{ role: "user", content: "Explain containerization" }],
    }),
  },
);
```

### Model Switching

```javascript
// Switch providers without changing code
const models = ["openai/gpt-4o-mini", "anthropic/claude-3-5-sonnet", "vertex/gemini-2.0-flash"];
const model = models[Math.floor(Math.random() * models.length)];

const response = await fetch(
  "https://www.nickhedberg.com/ai-api/v1/chat/completions",
  {
    // ... same headers and structure
    body: JSON.stringify({
      model: model, // Dynamic model selection
      messages: messages,
    }),
  },
);
```
