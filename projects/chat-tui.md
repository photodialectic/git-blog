
# Chat-TUI: Terminal AI Chat Interface

A Go-based terminal user interface for AI conversations built with Bubble Tea, providing a lightweight command-line alternative to web-based chat applications.

## Overview

After building the web-based [/chat-gpt](/chat-gpt) application, I wanted a terminal-native way to interact with AI models. Chat-TUI fills this gap by providing a fully featured chat interface that runs in the terminal with persistent configuration and seamless model switching. This also gave me an opportunity to validate an access-token authentication flow using Auth0.

![Chat-TUI Screenshot](https://www.nickhedberg.com/images/-Z84oPN3JyshiBiqaxIji5OOViU=/fit-in/1200x1200/s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1080%3A1516%2Fb1ded00e91eda1003a83119d783953a5ddba7d1c.png)

## Key Features

### Terminal-Native Interface
- **Bubble Tea Framework**: Built using Charm's Bubble Tea for rich terminal UIs
- **Color Support**: Full color support with automatic detection and theming
- **Mouse Support**: Optional mouse tracking for text selection
- **Keyboard Navigation**: Efficient keyboard-driven interface with intuitive shortcuts

### Chat Management
- **Persistent State**: Configuration and chat history saved to local JSON files
- **Chat Switching**: Browse and switch between multiple conversations with `/chat` command
- **Model Selection**: Interactive model picker with real-time switching via `/model` command
- **Multi-line Composition**: Dedicated compose mode for longer messages with `/compose`

### Configuration System
- **JSON Configuration**: Simple JSON-based config stored in `~/.config/chat-tui/config.json`
- **Token Management**: Secure token storage with Auth0 integration
- **Environment Variables**: Optional environment-based configuration override
- **Docker Support**: Containerized deployment with volume mounting for persistence

## Technical Implementation

### Go Architecture
```
internal/
├── api/          # HTTP client and streaming logic
│   ├── client.go
│   ├── stream.go
│   └── types.go
├── state/        # Application state management
│   └── state.go
└── render/       # Text rendering and formatting
    ├── markdown.go
    └── wrap.go

ui/               # Bubble Tea interface components
├── model.go      # Main application model
├── view.go       # UI rendering logic
├── update.go     # Event handling
└── commands.go   # Command processing
```

## Command System

### Built-in Commands
- `/chat [number]`: List and select conversations
- `/model [name]`: Interactive model selection or direct model setting
- `/compose`: Multi-line message composition mode
- `/config`: Display current configuration and token status
- `/exit`: Quit the application

### Authentication Flow
1. Visit the Auth0 token endpoint (`/auth/token`)
2. Log in and copy the provided access token
3. Paste the token into the TUI for automatic saving

## Development Experience

Building this project provided insights into:
- **Terminal UI Design**: Creating intuitive interfaces within terminal constraints
- **Go Concurrency**: Managing streaming responses and UI updates concurrently
- **State Management**: Balancing local persistence with real-time data
- **Cross-platform Deployment**: Docker containerization for consistent terminal environments

## Configuration Example

```json
{
  "token": "<ACCESS_TOKEN>",
  "api_base": "https://www.nickhedberg.com/chat-gpt",
  "model_id": "gpt-4o-mini",
  "chat_id": "optional-existing-chat-id"
}
```

## Docker Deployment

The application is fully containerized with sensible defaults:
- Color support enabled automatically
- Volume mounting for persistent state
- Host network access for API connectivity
- Environment variable overrides for theming

## Use Cases

Chat-TUI excels in scenarios where:
- **Server Administration**: Quick AI assistance while working on remote systems
- **Development Workflow**: Integrated AI help without leaving the terminal
- **Resource Efficiency**: Lightweight alternative to web interfaces
- **Automation**: Scriptable AI interactions for workflow integration

The combination of persistent state, efficient terminal rendering, and seamless API integration makes it a powerful tool for terminal-centric workflows.
