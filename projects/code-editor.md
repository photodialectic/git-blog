
# [Code-Editor: Browser-Based Development Environment](/code-editor)

A NextJS/React application that provides a full-featured code editor with AI assistance, live bundling, and real-time preview capabilities directly in the browser.

## Overview

Inspired by tools like CodeSandbox and Replit, this project creates a self-hosted development environment that runs entirely in the browser. It demonstrates advanced client-side bundling, AI integration, and real-time code execution without requiring server-side compilation.

I originally built this to explore agentic tools that can write and patch code. It's limited to editing just three files: HTML, CSS, and JS.

![Code Editor Screenshot](https://www.nickhedberg.com/images/AA1zIvgui1wjoDWFjjvzZNVXAO8=/fit-in/1200x1200/s3-us-west-2.amazonaws.com/nick-hedberg/img%2F2918%3A3006%2F4c2de7e989bbbfffddefb8e67e3ac12fad5421d6.png)

## Key Features

### Code Editing & Execution
- **Monaco Editor Integration**: Full VS Code editor experience with syntax highlighting and IntelliSense
- **Live Bundling**: Real-time JavaScript/TypeScript compilation using ESBuild WebAssembly (still a work in progress)
- **Module Resolution**: Automatic npm package fetching and dependency resolution
- **Preview System**: Instant code execution with isolated iframe rendering

### AI-Powered Development
- **Code Generation**: AI assistance for writing and refactoring code
- **Error Debugging**: AI-powered error analysis and suggestions
- **Code Explanation**: Natural language explanations of complex code segments
- **Multiple File Types**: Support for JavaScript, TypeScript, CSS, HTML, and JSON

### Project Management
- **File System**: Multi-file project support with virtual file system
- **Save & Share**: Persistent project storage with shareable URLs
- **Template System**: Pre-built templates for common project types
- **Export Options**: Download projects as ZIP files or individual files

## Technical Architecture

### Client-Side Bundling
```
lib/
├── bundler.js           # ESBuild WebAssembly integration
├── transpiler.js        # TypeScript compilation
└── plugins/
    ├── fetch-plugin.js      # HTTP module resolution
    └── unpkg-path-plugin.js # npm package resolution
```

The bundling system runs entirely in the browser using ESBuild's WebAssembly build, enabling real-time compilation without server round-trips.

### AI Integration Components
```
components/AIPromptBox/
├── AIPromptInterface.js    # Main AI interaction interface
├── AIPromptManager.js      # Request orchestration
├── AIDebugManager.js       # Error analysis and debugging
├── PromptModal.js          # AI prompt dialog
├── DebugWindow.js          # Error display and AI suggestions
└── ControlButtons.js       # AI action controls
```

### Code Execution System
The application uses a sandboxed iframe approach for code execution:
- **Isolation**: Each preview runs in a separate iframe context
- **Security**: CSP headers and sandboxing prevent malicious code execution
- **Real-time Updates**: Code changes trigger immediate re-bundling and preview updates
- **Error Handling**: Runtime errors captured and displayed with AI analysis

## Implementation Details

### ESBuild WebAssembly Integration
The core innovation is running ESBuild entirely in the browser:
- **Module Resolution**: Custom plugins resolve npm packages via unpkg CDN
- **Dependency Management**: Automatic detection and fetching of package dependencies
- **Performance**: Near-instant bundling for typical web projects
- **TypeScript Support**: Full TypeScript compilation without server-side processing

### AI Tool Integration
```javascript
// AI tools for code assistance
const aiTools = {
  generateCode: (prompt, fileType) => { /* AI code generation */ },
  debugError: (error, code) => { /* AI error analysis */ },
  explainCode: (codeSnippet) => { /* AI code explanation */ },
  refactorCode: (code, instructions) => { /* AI refactoring */ }
}
```

### File Management System
Virtual file system enabling multi-file projects:
- **In-Memory Storage**: Files stored in browser memory with persistence options
- **File Tree UI**: Visual file explorer with create/delete/rename operations
- **Import/Export**: Full project import/export functionality
- **Version Control**: Basic file history and change tracking

## Development Challenges

### Bundle Size Optimization
ESBuild WebAssembly is large (~2MB), requiring:
- **Lazy Loading**: ESBuild loaded only when needed
- **Caching Strategy**: Aggressive caching to avoid repeated downloads
- **Progressive Enhancement**: Basic editor functionality before bundler loads

### Module Resolution
Implementing npm module resolution in the browser required:
- **CDN Integration**: Seamless unpkg.com integration for package fetching
- **Dependency Trees**: Recursive dependency resolution and caching
- **Version Management**: Handling package version conflicts and updates

### Security Considerations
Running user code safely requires:
- **Iframe sandboxing**: Strict CSP policies for code execution contexts
- **API Limitations**: Restricted access to sensitive browser APIs
- **Resource Limits**: Prevention of infinite loops and memory exhaustion

## AI Agent Workspace

- **Session-Oriented Chats**: Every `/code-editor/api/ai` request goes through `mono/ai-agent-api`, so prompts create LiteLLM-backed sessions with persistent conversation history, tool-call tracking, and streaming support.
- **Tool Automation**: `useAIRequestService` inspects tool calls (read/write/replace/insert file helpers) and automatically feeds tool results back into the agent loop, letting the assistant diff CSS, refactor JS, or scaffold new files without copy/paste.
- **Local Chat Persistence**: `AIPromptManager` stores the full transcript, selected agent key, and session id in `localStorage`, which means the assistant resumes exactly where you left it when reopening the editor.

## Authenticated Workspaces & Token Budgets

- **Auth0 Role Gate**: The main page (`mono/code-editor/pages/index.js`) requires the `nickhedberg.com:code-editor` claim before exposing the workspace or AI API routes.
- **Token HUD**: A `TokenContext` fetches usage from `/code-editor/api/tokens`, rendering live token counts, estimated dollar cost, and a progress indicator so you know when you are nearing daily limits.
- **Quota Enforcement**: Each completion records input/output tokens in MySQL via `addTokenCount`, and `/code-editor/api/ai` blocks prompts when a user exceeds their `token_max` allowance.

## Persistent Projects & Sharing

- **Source Library**: `/code-editor/api/sources` persists each playground’s HTML/CSS/JS blob plus metadata, enabling save, fork, and soft-delete flows against the `source` table.
- **Router-Aware Loader**: `CodePlayground` watches the `[id]` query param, fetches the matching saved source, and hydrates the editors plus project name, making it easy to deep-link a specific demo.
- **Responsive UI Controls**: The editor automatically flips into a two-tab (Editor/Preview) layout on small screens, keeping Monaco usable on tablets and phones while still driving the same bundler pipeline.

The current stack combines client-side bundling, stateful AI assistance, and persistent projects to deliver a full browser-based development environment that feels surprisingly close to a local IDE.
