# [Code-Editor: Browser-Based Development Environment](/code-editor)

A Next.js/React app for editing HTML, CSS, and JavaScript in the browser, with live preview and an AI-assisted editing loop.

## Overview

This project is focused on fast browser-side iteration: edit three files (HTML/CSS/JS), see the result immediately in a sandboxed preview iframe, and optionally ask an AI agent to patch files for you.

![Code Editor Screenshot](https://www.nickhedberg.com/images/AA1zIvgui1wjoDWFjjvzZNVXAO8=/fit-in/1200x1200/s3-us-west-2.amazonaws.com/nick-hedberg/img%2F2918%3A3006%2F4c2de7e989bbbfffddefb8e67e3ac12fad5421d6.png)

## Key Features

### Editing and Preview
- **Monaco Editor**: Uses `@monaco-editor/react` for the editing panes.
- **Three-File Workflow**: Dedicated editors for HTML, CSS, and JS.
- **Live Preview**: JS is transpiled with Babel (`@babel/standalone`) and rendered in an isolated iframe.
- **Mobile-Friendly Layout**: On smaller screens, the UI switches between Editor and Preview tabs.

### AI-Assisted Workflow
- **Session-Based Agent Calls**: `/code-editor/api/ai` uses `mono/ai-agent-api` sessions for multi-turn context.
- **Tool Execution Loop**: The UI processes tool calls (`read/write/replace/insert` file helpers) and feeds tool results back automatically.
- **Streaming Support**: The AI request service supports streaming responses and tool status updates in-chat.
- **Local Session Persistence**: Session ID, selected agent key, and streaming preference are stored in `localStorage`.

### Persistence and Sharing
- **Saved Sources API**: `/code-editor/api/sources` supports create/update/load/delete of saved projects.
- **Route-Based Loading**: The `[id]` route param loads a saved project directly into the editor.
- **Project Naming**: Saved projects track and persist human-readable names.

## Technical Architecture

```text
mono/code-editor/
├── pages/index.js                    # Main authenticated app shell
├── components/CodePlayground.js      # Editor/preview orchestration
├── hooks/useCodeExecution.js         # Debounced transpile + preview pipeline
├── lib/transpiler.js                 # Babel transpilation helpers
├── pages/api/ai.js                   # AI request/session bridge
├── pages/api/sources/                # Project persistence endpoints
└── components/AIPromptBox/           # AI chat UI and request manager
```

## Auth and Usage Controls

- **Auth0 Sessions**: API routes require authenticated users.
- **Permission Gate**: `/code-editor/api/ai` enforces the `nickhedberg.com:code-editor` claim.
- **Cost Tracking HUD**: `/code-editor/api/tokens` powers the per-model usage/cost display in the header.
- **Daily Cost Limit**: AI route checks current usage and returns `429` when the configured per-user limit is exceeded.

The result is a compact browser IDE centered on quick iteration, persistent sharing, and an integrated AI editing assistant.
