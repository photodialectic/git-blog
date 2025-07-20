# AI Integration in HomeStack Services

With the AI-API providing unified access to multiple AI providers, integrating AI capabilities into HomeStack services becomes straightforward. Rather than managing different SDK versions and API keys, each service can simply point to the centralized AI endpoint.

## Integration Pattern

All AI-enabled services in my HomeStack follow the same integration pattern:

1. **Single dependency** - Use the OpenAI SDK (since AI-API presents OpenAI-compatible endpoints)
2. **Dynamic base URL** - Point to the AI-API instead of OpenAI directly
3. **Unified authentication** - Use the `AI_API_MK` master key
4. **Model selection** - Choose from any available model via string parameter

## Chat-GPT Service

My chat-gpt service demonstrates the most complete AI integration, providing a full conversational interface with model selection and function calling.

### API Configuration

```javascript
// chat-gpt/pages/api/chat/completions.js
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.AI_API_MK,
  baseURL: `${req.headers.origin}/ai-api`,
});
```

### Model Listing

```javascript
// chat-gpt/pages/api/models.js
export default withApiAuthRequired(async function handler(req, res) {
  const openai = new OpenAI({
    apiKey: process.env.AI_API_MK,
    baseURL: `${originalHost}/ai-api`,
  });

  const { data } = await openai.models.list();
  res.status(200).json({ data });
});
```

This seamlessly exposes all available models (GPT, Claude, Gemini) through a single endpoint, letting users switch between providers without the service caring about implementation details.

### Function Calling

The chat service also demonstrates AI function calling:

```javascript
const functionMap = {
  get_current_weather: {
    func: getCurrentWeather,
    config: {
      name: "get_current_weather",
      description: "Get the current weather in a given location",
      parameters: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "The city and state, e.g. San Francisco, CA",
          },
          unit: { type: "string", enum: ["celsius", "fahrenheit"] },
        },
        required: ["location"],
      },
    },
  },
};
```

This allows AI models to call JavaScript functions, extending their capabilities beyond text generation.

## Code-Editor Service

My code-editor service takes a different approach, using Anthropic's SDK directly but still routing through the AI-API for some requests.

### Anthropic Integration

```javascript
// code-editor/pages/api/ai.js
import Anthropic from "@anthropic-ai/sdk";

const api = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

### System Prompt Engineering

The code-editor service uses sophisticated system prompts to generate interactive web applications:

```javascript
const systemPrompt = `
You are an AI that creates interactive web games using HTML, CSS, and JavaScript.

PROCESS:
1. Determine which files need to be created or modified based on the user's request
2. Use the appropriate tools to write each required file (CSS, HTML, JavaScript)
3. Ensure all files work together correctly as a complete web application

COLOR PALETTE (use these for consistency):
- Primary colors: #ee3322 (red), #0f65ef (blue), #ffee00 (yellow)
- Accent colors: #eb5369 (pink), #b1ecfd (light blue), #f0fa81 (light green)

TECHNICAL REQUIREMENTS:
- Create responsive designs that work on both mobile and desktop
- Ensure all code is free of syntax errors
- Follow standard web development practices
- Include clear comments explaining key functionality
`;
```

This demonstrates how different services can have specialized AI behaviors while using the same underlying infrastructure.

## Authentication Integration

All AI-enabled services integrate with Auth0 authentication, ensuring users have appropriate permissions:

```javascript
// Permission checking in models.js
export default withApiAuthRequired(async function handler(req, res) {
  const { user } = await getSession(req, res);
  if (!user["nickhedberg.com:chat-gpt"]) {
    res.status(403).json({ message: "you need permissions" });
    return;
  }
  // ... proceed with AI API calls
});
```

This ensures AI capabilities are properly gated behind authentication and authorization.

## Docker Environment Integration

Services seamlessly discover the AI-API through Docker Compose environment variables:

```yaml
# docker-compose.yml
chat-gpt:
  environment:
    - AI_API_MK=${AI_API_MK}
    -  # other env vars...

code-editor:
  environment:
    - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    -  # other env vars...
```

The Traefik routing ensures services can reach the AI-API at predictable URLs (`/ai-api`), making the integration location-transparent.

## Benefits of This Architecture

This integration pattern provides several advantages:

### Development Simplicity

- **Single SDK** - Most services use OpenAI SDK regardless of actual provider
- **Consistent patterns** - Same authentication and URL structure across services
- **Easy testing** - Switch models by changing a string parameter

### Operational Benefits

- **Centralized logging** - All AI requests flow through one point
- **Cost visibility** - Track AI spending across all services
- **Rate limiting** - Implement usage controls at the API gateway level
- **Provider flexibility** - Switch providers without changing service code

### Security Advantages

- **Credential isolation** - Provider API keys stored in one secure location
- **Access control** - AI capabilities gated behind authentication
- **Audit trail** - Central logging of all AI interactions

## Real-World Usage

In practice, this architecture enables powerful AI features across the HomeStack:

- **Chat interface** with multiple model support and function calling
- **Code generation** with specialized prompts for web development
- **Content creation** tools that can switch between providers based on task
- **Development assistance** through Claudex integration

The unified AI-API acts as a force multiplier, making AI capabilities easily accessible to any service that needs them while maintaining security and operational simplicity.

## Future Possibilities

This foundation enables exciting future integrations:

- **Smart monitoring** - AI analysis of service logs and metrics
- **Automated deployment** - AI-assisted infrastructure management
- **Content optimization** - AI-powered SEO and performance analysis
- **User personalization** - AI-driven feature customization

The key insight is that by standardizing AI access through a single API, you remove the integration friction that typically prevents AI adoption in smaller-scale projects. Every service becomes AI-capable with minimal effort.
