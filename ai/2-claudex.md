# Claudex: Secure Containerized AI Development

Originally I built [Claudex](https://github.com/photodialectic/claudex) because I had security concerns about AI Agent CLIs like [Claude Code](https://claude.ai/code) and [Codex](https://codex.dev). These tools are incredibly powerful, but they also have the ability to read and write files, execute commands, and make network requests.

As these tools mature, some of them are mitigating these security risks and implementing features that give users more explicit control over risks like network and file system access. While I still have some concerns around security, that's becoming less of a reason for me to use Claudex. Instead I find Claudex useful because it’s very portable, I can control its context very explicitly, and it allows me to stay in my terminal and use my own editor and tools.

I use Claudex locally and on remote servers to plan, debug, and develop new features.

## Claudex Architecture

Claudex addresses these concerns through containerization with strict isolation:

```bash
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
- **Container sessions** with naming, parallel mode, list/destroy management

## Installation and Setup

Claudex is distributed as a Go CLI that manages Docker containers:

```bash
# Install from source
git clone https://github.com/photodialectic/claudex.git
cd claudex
make install

# Build the container image
claudex build
```

## Usage Examples

### Basic Project Development

```bash
# Start a session mounting the current directory
claudex

# Mount multiple services into one session
claudex service1/ service2/ service3/

# Name a session, or force a parallel session
claudex --name my-api ./api
claudex --parallel ./app ./api
```

### Push a spec or screenshot

```bash
claudex push SPEC.md screenshot.png
# Files are copied into /workspace inside the running container
```

### Pull files created by the AI

I often use my prod server to run claudex and an agent in read-only to develop a feature plan remotely.

The agent may create artifacts in `/workspace` (outside mounted project directories). Use `claudex pull` to copy them to your host.

Note: the destination is a directory. Use `.` to copy into the current directory, or provide an output folder.

```bash
# Copy SPEC.md from the container to the current directory
claudex pull /workspace/SPEC.md .

# Or copy a folder into an output directory
claudex pull /workspace/output ./out
```

I will often use this local file to create an issue in my project's git repo using the GitHub CLI:

```bash
gh issue create --title "New Feature" --body-file SPEC.md
```

### Network Access
Sometimes you want to allow network access for authentication flows (like GitHub OAuth). You can enable this with the `--host-network` flag:

```bash
# Allow network access for authentication flows
claudex --host-network
```

### Manage Sessions

You can manage containers created by Claudex:

```bash
# List sessions (running by default)
claudex list

# List all, or only stopped
claudex list --all
claudex list --stopped

# Destroy by name, by signature, or prune stopped
claudex destroy --name my-api
claudex destroy --signature <HASH>
claudex destroy --prune-stopped
```

## Container Environment

Inside the Claudex container, you get a fully configured development environment:

### File System Layout

```bash
/workspace/
├── service1/          # Your mounted projects
├── service2/
├── .git/              # Auto-initialized Git repo
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
- Not intended for anything other than local tracking of changes

## AI Integration with HomeStack

Claudex integrates seamlessly with my AI-API through configuration:

### Claude Code Configuration

```json
// ~/.claude.json
{
  "apiUrl": "https://www.nickhedberg.com/ai-api",
  "apiKey": "${AI_API_MK}",
  "model": "claude-4"
}
```

### Codex Configuration

```toml
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

```json
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

```json
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
