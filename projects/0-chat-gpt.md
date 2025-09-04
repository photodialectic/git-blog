# [Chat-GPT: AI Chat Application](/chat-gpt)

A NextJS/React web application that provides a clean interface for AI conversations with multiple model support and persistent chat history.

## Overview

I built this application in the summer of 2023 to help me explore OpenAI's API and play around with AI APIs in general. I use it daily as a alternative to ChatGPT for quick questions and brainstorming.

I continue to add features and improvements like tool calling, model switching backed by my self-hosted LiteLLM reverse proxy and my AI-API service, vision, code formatting, copy buttons, etc.

## Key Features

### Multi-Model Support
- **Model Selection**: Switch between different AI models (GPT-4, GPT-4o-mini, etc.) within the same chat
- **Provider Integration**: Uses my AI-API service as a unified backend for multiple AI providers
- **Dynamic Configuration**: Model availability and settings managed server-side

### Chat Management
- **Persistent History**: All conversations saved to MySQL database with user association
- **Auto-Save**: Chat content automatically saved as you type using custom React hooks
- **Chat Organization**: Browse and resume previous conversations
- **Real-time Streaming**: Responses stream in real-time for better user experience

## Technical Architecture

### Frontend Stack
- **NextJS**: React framework with API routes for backend integration
- **Custom Hooks**:
  - `useChatAutoSave`: Automatic saving of chat content
  - `useChatData`: Chat history and message management
  - `useChatSettings`: User preferences and model selection
  - `useChatStream`: Real-time message streaming
  - `useStoredConvos`: Local storage integration

### Backend Integration
- **AI-API Connection**: Interfaces with my self-hosted LiteLLM reverse proxy
- **Database Layer**: MySQL for chat persistence and user data
- **Authentication**: Auth0 SDK for secure user sessions

### Component Architecture
```
components/
├── ChatHistoryModal.js    # Chat browsing and selection
├── ErrorBoundary.js       # Error handling wrapper
├── chat.js               # Main chat interface
├── functions.js          # Tool calling and function execution
├── history.js            # Chat history sidebar
├── login.js              # Authentication components
├── main.js               # Layout and navigation
├── menu.js               # Settings and model selection
└── message.js            # Individual message rendering
```

## Implementation Details

### Streaming Chat Interface
The application handles real-time AI responses through a streaming API endpoint that processes chunks of data as they arrive, updating the UI incrementally for a smooth conversation experience.

### Auto-Save Functionality
Custom React hooks automatically save chat content to prevent data loss, with debounced saving to avoid excessive database writes while maintaining responsiveness.

### Tool Integration
The chat interface supports AI tool calling, allowing the AI to execute functions and integrate external data sources during conversations.
