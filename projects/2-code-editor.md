# Code-Editor: Browser-Based Development Environment

A NextJS/React application that provides a full-featured code editor with AI assistance, live bundling, and real-time preview capabilities directly in the browser.

## Overview

Inspired by tools like CodeSandbox and Repl.it, this project creates a self-hosted development environment that runs entirely in the browser. It demonstrates advanced client-side bundling, AI integration, and real-time code execution without requiring server-side compilation.

## Key Features

### Code Editing & Execution
- **Monaco Editor Integration**: Full VS Code editor experience with syntax highlighting and IntelliSense
- **Live Bundling**: Real-time JavaScript/TypeScript compilation using ESBuild WebAssembly
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
- **Iframe Sandboxing**: Strict CSP policies for code execution contexts
- **API Limitations**: Restricted access to sensitive browser APIs
- **Resource Limits**: Prevention of infinite loops and memory exhaustion

## Use Cases

The code editor excels for:
- **Rapid Prototyping**: Quick idea testing without local environment setup
- **Learning & Teaching**: Interactive code examples and tutorials
- **Code Sharing**: Easily shareable working code examples
- **AI-Assisted Development**: Learning new APIs with AI guidance

## Performance Characteristics

- **Initial Load**: ~3-5 seconds for full editor with bundling capability
- **Bundle Time**: <500ms for typical projects under 100KB
- **Memory Usage**: ~50-100MB depending on project complexity
- **Network**: Minimal after initial load, only fetching new npm packages

The combination of client-side bundling, AI assistance, and real-time preview creates a powerful development environment that runs entirely in the browser while maintaining near-native performance.