# [Workspace-API: Docker Workspace Management Service](/docs/oas/workspace-api.yml)

A Go-based API service for managing isolated Docker container workspaces with integrated terminal access, persistent storage, and real-time communication via WebSocket.

## Overview

Workspace-API is the backend orchestrator for containerized development environments. It manages workspace lifecycle (creation, execution, teardown), handles file I/O operations, manages terminal sessions via WebSocket, and integrates with Redis for caching and Docker for container orchestration.

## Key Features

### Workspace Orchestration
- **Container Lifecycle Management**: Create, run, and terminate isolated Docker workspaces
- **Dynamic Recovery**: Automatic recovery of running workspaces on service startup
- **Resource Isolation**: Each workspace runs in its own container with independent filesystem and network

### Terminal Management
- **WebSocket Terminal Access**: Real-time terminal sessions via `/workspace/:id/terminal`
- **JWT-Based Authentication**: Secure terminal connections with token-based auth
- **Session Persistence**: Store and manage terminal session tokens in Redis

### Data Persistence
- **MySQL Integration**: Persistent workspace metadata and configuration storage
- **File Operations**: Direct file I/O for workspace content management
- **Caching Layer**: Redis-backed caching for performance optimization

### Docker Integration
- **Claudex Support**: Optional Claudex container image building and management
- **Container Health**: Automatic recovery and health monitoring of running containers
- **Docker Socket Access**: Direct Docker API calls for full container control

## Technical Architecture

```
workspace-api/
├── handlers/       # HTTP handlers for workspace CRUD and terminal operations
├── internal/
│   ├── database/   # MySQL connection and schema management
│   ├── docker/     # Docker client wrapper and container operations
│   ├── terminal/   # Terminal session management and WebSocket handling
│   └── redis/      # Redis client for caching and session storage
├── store/          # Data access layer for workspace state
├── docker/         # Docker utilities and container configuration
└── main.go         # Router wiring and dependency injection
```

## API Endpoints

### Workspace Management
- `POST /workspace` - Create new workspace
- `GET /workspace/:id` - Get workspace details
- `DELETE /workspace/:id` - Terminate workspace
- `POST /workspace/:id/exec` - Execute commands in workspace

### Terminal Operations
- `POST /workspace/:id/terminal-token` - Request terminal access token
- `WS /workspace/:id/terminal` - WebSocket terminal connection

## Dependencies

- **Docker SDK**: Container management and orchestration
- **MySQL**: Persistent workspace storage
- **Redis**: Session caching and terminal token management
- **Gorilla WebSocket**: Real-time terminal communication
- **JWT**: Secure terminal authentication
