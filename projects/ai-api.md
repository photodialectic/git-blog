# [AI-API: Self-Hosted Bifrost Gateway](/docs/oas/ai-api.yml)

A unified AI gateway that consolidates multiple AI providers (OpenAI, Anthropic, Gemini, and Vertex) behind a single API endpoint with centralized authentication and governance.

## Overview

As AI capabilities became central to my HomeStack projects, managing multiple API keys and provider-specific request formats became unwieldy. AI-API solves this by exposing a single OpenAI-compatible interface, backed by a Bifrost config that maps providers and virtual keys in one place.

![AI-API Screenshot](https://www.nickhedberg.com/images/CBn_eMPwYruDs_uKNkpdnOCWZ5E=/fit-in/1200x1200/nhdc.nyc3.cdn.digitaloceanspaces.com/img%2F1472%3A2430%2F7691c4ccd4669c1e9f1b0227fcd64fa8168069eb.png)

## Key Features

### Provider Unification
- **Multi-Provider Support**: OpenAI, Anthropic Claude, Google Gemini, and others
- **Standardized Interface**: Single OpenAI-compatible API for all providers

## Technical Architecture

### Bifrost Integration
```yaml
# Core configuration shape (mono/ai-api/config.json)
providers:
  openai:
    keys:
      - name: openai-primary
        value: env.OPENAI_API_KEY
  anthropic:
    keys:
      - name: anthropic-primary
        value: env.ANTHROPIC_API_KEY
  gemini:
    keys:
      - name: gemini-primary
        value: env.GEMINI_API_KEY
governance:
  virtual_keys:
    - name: master
      value: env.AI_API_MK
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
  apiKey: process.env.AI_API_MK,
  baseURL: "https://www.nickhedberg.com/ai-api/v1",
});
```
