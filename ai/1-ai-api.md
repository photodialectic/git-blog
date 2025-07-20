# AI-API: Self-Hosted LiteLLM Reverse Proxy

Managing multiple AI provider APIs (OpenAI, Anthropic, Google) across different services becomes unwieldy quickly. Each service needs its own API keys, rate limiting, and provider-specific code. My solution: a unified AI-API using LiteLLM as a reverse proxy.

## Why LiteLLM?

[LiteLLM](https://docs.litellm.ai/) translates requests between different AI provider APIs, presenting a unified OpenAI-compatible interface. This means:

- **Single integration point** - all services use the same API format
- **Provider abstraction** - switch between models without changing application code
- **Centralized authentication** - one master key instead of multiple provider keys
- **Built-in features** - logging, rate limiting, cost tracking, and fallbacks

## Configuration

My AI-API runs as a Docker container in the HomeStack with this LiteLLM configuration:

```language-yaml
model_list:
  # OpenAI Models
  - model_name: gpt-4o-mini
    litellm_params:
      model: openai/gpt-4o-mini
      api_key: os.environ/OPENAI_API_KEY

  - model_name: gpt-3.5
    litellm_params:
      model: openai/gpt-3.5-turbo
      api_key: os.environ/OPENAI_API_KEY

  # Anthropic Models
  - model_name: claude-4
    litellm_params:
      model: anthropic/claude-sonnet-4-20250514
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: claude-3-5-sonnet
    litellm_params:
      model: anthropic/claude-3-5-sonnet-20241022
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: claude-3-5-haiku
    litellm_params:
      model: anthropic/claude-3-5-haiku-20241022
      api_key: os.environ/ANTHROPIC_API_KEY

  # Google Models
  - model_name: gemini-2.0-flash
    litellm_params:
      model: gemini/gemini-2.0-flash
      api_key: os.environ/GEMINI_API_KEY

  # Image Generation
  - model_name: dalle3
    litellm_params:
      model: openai/dall-e-3
      api_key: os.environ/OPENAI_API_KEY

general_settings:
  master_key: os.environ/AI_API_MK
  disable_spend_logs: True
  disable_error_logs: True

litellm_settings:
  log_raw_request_response: False
```

## Docker Integration

The AI-API service integrates seamlessly into my Docker Compose stack:

```language-yaml
ai-api:
  container_name: ai-api
  image: ghcr.io/berriai/litellm:main-latest
  environment:
    - OPENAI_API_KEY=${OPENAI_API_TOKEN}
    - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    - GEMINI_API_KEY=${GEMINI_API_KEY}
    - AI_API_MK=${AI_API_MK}
    - SERVER_ROOT_PATH=/ai-api
  labels:
    - traefik.enable=true
    - traefik.http.services.ai-api.loadbalancer.server.port=10200
    - traefik.http.routers.ai-api.rule=Host(`www.nickhedberg.com`) && PathPrefix(`/ai-api`)
    - traefik.http.routers.ai-api.entrypoints=websecure
    - traefik.http.routers.ai-api.tls.certresolver=wwwresolver
  ports:
    - 10200:10200
  volumes:
    - ./ai-api/config.yml:/app/config.yml
```

Key configuration details:

- **Path-based routing** - accessible at `www.nickhedberg.com/ai-api`
- **Environment variables** - provider API keys loaded from encrypted secrets
- **Volume mount** - configuration file mounted into container
- **Master key authentication** - single key controls access to all models

## Usage Examples

Services can now use any AI model through a single endpoint:

### Chat Completion

```language-javascript
const response = await fetch(
  "https://www.nickhedberg.com/ai-api/chat/completions",
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${AI_API_MK}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "claude-4",
      messages: [{ role: "user", content: "Explain containerization" }],
    }),
  },
);
```

### Model Switching

```language-javascript
// Switch providers without changing code
const models = ["gpt-4o-mini", "claude-3-5-sonnet", "gemini-2.0-flash"];
const model = models[Math.floor(Math.random() * models.length)];

const response = await fetch(
  "https://www.nickhedberg.com/ai-api/chat/completions",
  {
    // ... same headers and structure
    body: JSON.stringify({
      model: model, // Dynamic model selection
      messages: messages,
    }),
  },
);
```

## Benefits in Practice

This setup has transformed how I integrate AI across HomeStack services:

1. **Simplified integration** - every service uses the same API contract
2. **Centralized cost tracking** - all AI spend flows through one endpoint
3. **Easy model experimentation** - switch models by changing a string
4. **Provider redundancy** - if one provider is down, switch to another
5. **Security** - provider API keys stored in one secure location

The AI-API acts as the unified brain of my HomeStack, making AI capabilities easily accessible to any service that needs them. It's the foundation that makes sophisticated AI integration practical at the HomeStack scale.
