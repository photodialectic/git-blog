# Claudex: Secure Containerized AI Development

AI agents like Claude Code and Codex are powerful development tools, but they come with security concerns. They can execute commands, access files, and make network requests. Claudex solves this by providing a secure, isolated Docker environment specifically designed for AI agent development.

## The Security Problem

AI agents need broad system access to be useful, but this creates risks:

- **File system access** - agents can read sensitive files or modify critical code
- **Network access** - agents might leak data or download malicious content
- **Command execution** - agents can run potentially dangerous system commands
- **Workspace persistence** - no tracking of what changes agents make

## Claudex Architecture

Claudex addresses these concerns through containerization with strict isolation:

```
┌─────────────────────────────────────────┐
│ Host System                             │
│  ┌─────────────────────────────────────┐│
│  │ Docker Container (Claudex)          ││
│  │  ┌─────────────────────────────────┐││
│  │  │ AI Agent (claude-code/codex)    │││
│  │  │  - Restricted network access    │││
│  │  │  - Git workspace tracking       │││
│  │  │  - Mounted project files        │││
│  │  └─────────────────────────────────┘││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

**Key Features:**

- **Network isolation** via iptables firewall
- **File system sandboxing** through selective mounting
- **Git-based change tracking** for all modifications
- **Pre-installed AI CLIs** (claude-code, codex)

## Installation and Setup

Claudex is distributed as a Go CLI that manages Docker containers:

```language-bash
# Install from source
git clone https://github.com/photodialectic/claudex.git
cd claudex
./install /usr/local/bin

# Build the container image
claudex build
```

The Dockerfile creates a secure environment:

```language-dockerfile
FROM ubuntu:22.04

# Install development tools
RUN apt-get update && apt-get install -y \
    git curl wget \
    ripgrep fd-find jq fzf tree \
    iptables sudo

# Install AI CLIs
RUN curl -sSL https://claude.ai/install | sh
RUN curl -sSL https://codex.dev/install | sh

# Create isolated user
RUN useradd -m -s /bin/bash node
USER node
WORKDIR /workspace

# Initialize firewall on container start
COPY init-firewall.sh /init-firewall.sh
ENTRYPOINT ["/init-firewall.sh"]
```

## Usage Examples

### Basic Project Development

```language-bash
# Mount current project and start AI session
claudex

# Mount multiple services
claudex service1/ service2/ service3/
```

### With Instructions Context

```language-bash
# Include project documentation for AI context
claudex --include ./docs
```

### Network Access for OAuth

```language-bash
# Allow network access for authentication flows
claudex --host-network
```

## Container Environment

Inside the Claudex container, you get a fully configured development environment:

### File System Layout

```
/workspace/
├── service1/          # Your mounted projects
├── service2/
├── .git/              # Auto-initialized Git repo
└── instructions/      # Optional context files

/context/              # Files added via --include
```

### Network Security

The firewall blocks most outbound connections but allows:

- Essential package management (apt, npm, pip)
- Git operations to known repositories
- AI API endpoints (claude.ai, openai.com)

### Git Integration

Every Claudex session automatically:

- Initializes a Git repository in `/workspace`
- Commits initial state of all mounted files
- Tracks all changes made by AI agents
- Provides full audit trail of modifications

## AI Integration with HomeStack

Claudex integrates seamlessly with my AI-API through configuration:

### Claude Code Configuration

```language-json
// ~/.claude.json
{
  "apiUrl": "https://www.nickhedberg.com/ai-api",
  "apiKey": "${AI_API_MK}",
  "model": "claude-4"
}
```

### Codex Configuration

```language-toml
# ~/.codex/config.toml
[model_providers.nhdc_ai_api]
name = "HomeStack AI API"
base_url = "https://www.nickhedberg.com/ai-api"
env_key = "AI_API_MK"

[profiles.homestack]
model_provider = "nhdc_ai_api"
model = "claude-3-5-sonnet"
```

This routes all AI requests through my centralized AI-API instead of directly to providers.

## Advanced Features

### MCP Server Integration

Claudex supports Model Context Protocol (MCP) servers running in Docker:

```language-json
// ~/.claude.json
{
  "mcpServers": {
    "fetch": {
      "command": "sudo",
      "args": ["docker", "run", "-i", "--rm", "mcp/fetch"]
    }
  }
}
```

### Agentception: Codex as MCP Server

You can even run Codex as an MCP server within Claude:

```language-json
{
  "mcpServers": {
    "codex": {
      "command": "codex",
      "args": ["mcp"]
    }
  }
}
```

This creates a fascinating AI inception where Claude can call Codex as a tool!

## Security Benefits

Claudex provides multiple layers of security:

1. **Process isolation** - AI agents can't escape the container
2. **Network restrictions** - firewall prevents data exfiltration
3. **File system limits** - only mounted directories are accessible
4. **Audit trail** - Git tracks every change made
5. **Ephemeral sessions** - containers are destroyed after use

## Development Workflow

A typical Claudex session looks like:

```language-bash
# Start secure AI development environment
claudex ./my-project

# Inside container - AI agents have access to:
# - Project files (read/write)
# - Development tools (git, npm, etc)
# - AI APIs (through controlled network access)
# - But NOT sensitive host system files

# All changes are tracked in Git
git log --oneline  # See what the AI agent modified

# Commit and exit when done
git commit -m "AI-assisted feature implementation"
exit
```

## Why This Matters

Claudex transforms AI-assisted development from a security risk into a secure, auditable process. It gives you the power of AI agents while maintaining:

- **Control** over what systems agents can access
- **Visibility** into what changes agents make
- **Security** through multiple isolation layers
- **Flexibility** to work with any AI provider or model

This is essential infrastructure for safely incorporating AI agents into serious development workflows. You get the productivity benefits without compromising security.
