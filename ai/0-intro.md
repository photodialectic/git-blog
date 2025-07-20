# AI in HomeStack

Artificial intelligence has become an integral part of my HomeStack infrastructure. Rather than relying on individual AI service subscriptions or managing multiple API keys across different applications, I've built a unified AI layer that serves all my projects.

This series covers how I've integrated AI capabilities into my service-oriented architecture, focusing on practical implementation rather than theoretical concepts.

## Architecture Overview

My AI setup consists of two main components:

1. **AI-API**: A self-hosted LiteLLM reverse proxy that unifies multiple AI providers behind a single endpoint
2. **Claudex**: A secure, containerized development environment for AI agents

This architecture provides several key benefits:

- **Single API endpoint** for all AI providers (OpenAI, Anthropic, Google)
- **Unified authentication** using master keys instead of individual provider keys
- **Cost tracking and rate limiting** across all AI usage
- **Secure development environment** for AI agent experimentation

## Posts

[1. AI-API: Self-Hosted LiteLLM Reverse Proxy](/blog/ai/1-ai-api.md)

How I unified multiple AI providers (OpenAI, Anthropic, Google) behind a single API endpoint using LiteLLM, with centralized authentication and cost tracking.

[2. Claudex: Secure Containerized AI Development](/blog/ai/2-claudex.md)

A Docker-based environment for running AI agents like Claude Code and Codex with strict network isolation and Git-based workspace tracking.

[3. AI Integration in HomeStack Services](/blog/ai/3-integration.md)

Practical examples of how AI capabilities are embedded across my HomeStack services, from code generation to content creation.
