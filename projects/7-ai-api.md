# AI-API: Self-Hosted LiteLLM Reverse Proxy

A unified AI gateway that consolidates multiple AI providers (OpenAI, Anthropic, Google) behind a single API endpoint with centralized authentication, cost tracking, and rate limiting.

## Overview

As AI capabilities became central to my HomeStack projects, managing multiple API keys, different request formats, and cost tracking across providers became unwieldy. AI-API solves this by creating a unified interface that abstracts away provider-specific implementations while adding observability and control layers.

## Key Features

### Provider Unification
- **Multi-Provider Support**: OpenAI, Anthropic Claude, Google Gemini, and others
- **Standardized Interface**: Single OpenAI-compatible API for all providers
- **Automatic Fallback**: Provider failover for high availability
- **Load Balancing**: Distribute requests across multiple provider endpoints

### Authentication & Security
- **Master Key System**: Single authentication token for all AI services
- **Request Validation**: Input sanitization and request verification
- **Rate Limiting**: Configurable limits per user and per provider
- **Usage Tracking**: Comprehensive logging of all AI requests and responses

### Cost Management
- **Real-Time Tracking**: Token usage and cost monitoring across providers
- **Budget Controls**: Automatic request limiting when budgets are exceeded
- **Provider Optimization**: Automatic routing to most cost-effective providers
- **Detailed Reporting**: Cost breakdowns by service, user, and time period

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

### Request Processing Pipeline
```python
# Simplified request flow
async def process_ai_request(request):
    # 1. Authentication validation
    user = authenticate_request(request.headers.authorization)
    
    # 2. Rate limiting check
    check_rate_limits(user, request.model)
    
    # 3. Cost validation
    validate_budget_limits(user, estimated_cost)
    
    # 4. Provider routing
    response = await route_to_provider(request)
    
    # 5. Usage tracking
    log_usage(user, request, response, actual_cost)
    
    return response
```

### High Availability Design
- **Health Checks**: Continuous monitoring of provider endpoint availability
- **Circuit Breakers**: Automatic provider failover during outages
- **Request Queuing**: Graceful handling of traffic spikes
- **Graceful Degradation**: Fallback to lower-cost models during provider issues

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

### HomeStack Service Integration
All HomeStack services benefit from unified AI access:
- **Chat-GPT**: Model switching without code changes
- **Code-Editor**: AI assistance with consistent interface
- **Documentation**: Content generation across different models
- **Chores**: AI-powered task suggestions and optimization

## Operational Features

### Monitoring & Observability
- **Request Metrics**: Response times, error rates, and throughput
- **Cost Analytics**: Real-time spend tracking and forecasting
- **Model Performance**: Comparative analysis across providers
- **Usage Patterns**: Understanding how different services use AI

### Configuration Management
- **Hot Reloading**: Configuration updates without service restart
- **Environment-Specific Settings**: Different configs for dev/prod
- **Feature Flags**: Gradual rollout of new providers or models
- **A/B Testing**: Provider performance comparison with real traffic

### Error Handling & Resilience
```python
# Comprehensive error handling
class AIAPIException(Exception):
    def __init__(self, provider, error_type, message, retry_after=None):
        self.provider = provider
        self.error_type = error_type  # rate_limit, quota, provider_down, etc.
        self.retry_after = retry_after
        super().__init__(message)

# Automatic retry with exponential backoff
async def robust_ai_request(request, max_retries=3):
    for attempt in range(max_retries):
        try:
            return await make_request(request)
        except AIAPIException as e:
            if e.error_type == 'rate_limit' and attempt < max_retries - 1:
                await asyncio.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
```

## Performance Optimization

### Caching Strategy
- **Response Caching**: Cache identical requests to reduce provider calls
- **Model Metadata**: Cache model capabilities and pricing information
- **Configuration Caching**: In-memory config to avoid repeated file reads
- **Smart Invalidation**: Cache invalidation based on request patterns

### Request Optimization
- **Batch Processing**: Combine multiple requests where supported
- **Connection Pooling**: Persistent connections to provider APIs
- **Request Deduplication**: Avoid duplicate requests in flight
- **Streaming Support**: Real-time response streaming for better UX

## Cost Optimization Features

### Intelligent Routing
```python
# Cost-aware model selection
def select_optimal_model(request, requirements):
    models = get_available_models(request.capabilities)
    
    # Filter by quality requirements
    suitable_models = [m for m in models if meets_quality_threshold(m, requirements)]
    
    # Select lowest cost option
    return min(suitable_models, key=lambda m: m.cost_per_token)
```

### Budget Management
- **Service-Level Budgets**: Different spending limits per HomeStack service
- **Time-Based Limits**: Daily, weekly, and monthly budget controls
- **Alert System**: Notifications when approaching budget limits
- **Automatic Throttling**: Gradual request limiting as budgets near limits

## Development Experience

### Local Development
- **Mock Responses**: Offline development with cached AI responses
- **Cost Simulation**: Test budget and rate limiting without real costs
- **Provider Simulation**: Test failover scenarios locally
- **Request Replay**: Debug issues by replaying production requests

### Testing Strategy
- **Unit Tests**: Individual component testing with mocked providers
- **Integration Tests**: End-to-end testing with real provider APIs
- **Load Testing**: Performance validation under various traffic patterns
- **Chaos Engineering**: Intentional provider failures to test resilience

## Security Considerations

### API Key Management
- **Rotation Support**: Seamless API key rotation without downtime
- **Principle of Least Privilege**: Minimal required permissions for provider keys
- **Audit Logging**: Complete audit trail of all API key usage
- **Secure Storage**: Encrypted storage of sensitive configuration data

### Request Sanitization
- **Input Validation**: Strict validation of all incoming requests
- **Content Filtering**: Optional content filtering for sensitive applications
- **Request Size Limits**: Protection against oversized requests
- **Output Sanitization**: Clean AI responses before returning to clients

## Future Enhancements

### Advanced Features
- **Custom Model Training**: Integration with fine-tuning workflows
- **Multi-Modal Support**: Support for image, audio, and video models
- **Federated Learning**: Privacy-preserving model improvement
- **Edge Deployment**: CDN-based AI inference for reduced latency

### Analytics & Intelligence
- **Usage Prediction**: Forecast AI spending and capacity needs
- **Quality Metrics**: Automated evaluation of AI response quality
- **A/B Testing**: Systematic comparison of different models and providers
- **Performance Optimization**: ML-based routing and caching decisions

AI-API serves as the intelligent foundation that makes AI capabilities seamlessly available across all HomeStack services while providing the operational controls necessary for production deployment.