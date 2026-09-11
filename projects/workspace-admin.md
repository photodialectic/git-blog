# [Workspace-Admin: Workspace Management UI](/docs/oas/workspace-admin.yml)

A React + Vite web application that provides a user interface for managing isolated Docker workspaces, complete with integrated terminal access and workspace configuration.

## Overview

Workspace-Admin is the frontend for the Workspace-API, enabling users to create, manage, and interact with isolated development environments. Built with React 18, Vite, and xterm.js, it provides a modern interface for terminal access, workspace configuration, and real-time operations.

## Key Features

### Workspace Management UI
- **Workspace Creation**: Simplified interface for spinning up new isolated environments
- **Workspace Listing**: Browse and manage active workspaces
- **Configuration Management**: Adjust workspace settings and resource allocation

### Integrated Terminal
- **xterm.js Integration**: Full-featured terminal emulator in the browser
- **WebSocket Connection**: Real-time terminal communication with Workspace-API
- **Auto-connection**: Automatic terminal attachment to selected workspace

### Developer Experience
- **Vite Hot Module Replacement (HMR)**: Fast development feedback loop
- **Express Server**: Dual-mode operation (dev with HMR, production serving)
- **Consistent API Routing**: Unified API route handling in both dev and prod

### Authentication
- **Auth0 Integration**: Enterprise-grade authentication via Auth0
- **Session Management**: Secure workspace access control

## Technical Architecture

```
workspace-admin/
├── src/
│   ├── components/     # React UI components for workspace management
│   ├── pages/          # Page-level components
│   ├── hooks/          # Custom React hooks
│   └── App.jsx         # Root application component
├── server/
│   ├── index.js        # Production Express server
│   ├── dev.js          # Development server with Vite middleware
│   └── routes.js       # Shared API route handlers
├── vite.config.js      # Vite configuration with React plugin
└── package.json        # Dependencies and scripts
```

## Available Scripts

- `npm install` — Install dependencies
- `npm run dev` — Start Express + Vite dev server with HMR (port 10131)
- `npm run build` — Build optimized production bundle
- `npm run preview` — Serve production build locally
- `npm run start` — Run Express server with built app

## Integration Points

### Workspace-API Communication
- **Base URL**: Configurable via `VITE_WORKSPACE_API_BASE` env var (defaults to `/workspace-api`)
- **Terminal Endpoint**: WebSocket connection to `/workspace-api/workspace/:id/terminal`
- **Token Management**: Requests terminal tokens via `POST /workspace/:id/terminal-token`

### Deployment Configuration
- **App Path**: Served under `/workspace-admin/` route (configured in vite.config.js)
- **Allowed Hosts**: Configured for local.dev, dev, stage, and production domains
- **Container Setup**: Docker volume for `/app/node_modules` to avoid bind mount shadowing

## Technology Stack

- **Frontend**: React 18.3, React Router 6
- **Build Tool**: Vite 5.2
- **Terminal**: xterm.js 6.0
- **Server**: Express.js 4.19
- **Authentication**: Auth0
