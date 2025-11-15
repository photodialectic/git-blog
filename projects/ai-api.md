# [AI-API: Self-Hosted LiteLLM Reverse Proxy](/docs/oas/ai-api.yml)

A unified AI gateway that consolidates multiple AI providers (OpenAI, Anthropic, Google) behind a single API endpoint with centralized authentication, cost tracking, and rate limiting.

## Overview

As AI capabilities became central to my HomeStack projects, managing multiple API keys, different request formats, and cost tracking across providers became unwieldy. AI-API solves this by creating a unified interface that abstracts away provider-specific implementations while adding observability and control layers.

![AI-API Screenshot](https://www.nickhedberg.com/images/lk5BCDIikGZrClW1vf2bIVtsKtI=/fit-in/1200x1200/https://s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1794%3A2172%2F0a4a038d73233d622e1c89c29034014544eb94af.png)

## Key Features

### Provider Unification
- **Multi-Provider Support**: OpenAI, Anthropic Claude, Google Gemini, and others
- **Standardized Interface**: Single OpenAI-compatible API for all providers

## Technical Architecture

### LiteLLM Integration
```yaml
# Core configuration structure
model_list:
  - model_name: gpt-4
    litellm_params:
      model: openai/gpt-4
      api_key: env/OPENAI_API_KEY
  - model_name: claude-3-sonnet
    litellm_params:
      model: anthropic/claude-3-sonnet-20240229
      api_key: env/ANTHROPIC_API_KEY
  - model_name: gemini-pro
    litellm_params:
      model: gemini/gemini-pro
      api_key: env/GOOGLE_API_KEY
```

## Integration Benefits

### Simplified Client Code
Instead of managing multiple SDKs and authentication methods:
```javascript
// Before: Multiple provider implementations
const openai = new OpenAI({ apiKey: process.env.OPENAI_KEY });
const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_KEY });

// After: Single unified interface
const ai = new OpenAI({
  apiKey: process.env.AI_API_KEY,
  baseURL: 'https://nickhedberg.com/ai-api'
});
```
