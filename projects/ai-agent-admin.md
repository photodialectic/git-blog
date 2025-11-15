# [AI-Agent-Admin: Operator Console](/ai-agent-admin)

A Next.js 15 application that gives operators a secure UI for managing agents, reviewing conversations, and replaying sessions powered by `mono/ai-agent-api`.

## Overview

AI-Agent-Admin is the control room for my conversational stack. It authenticates through Auth0, enforces role-based access, and proxies every request through the same host so the UI never exposes raw API keys. From here I can create/edit agents, audit tool calls, filter conversations by agent, and inspect session state without touching the database.

![AI Agent Admin Screenshot](https://www.nickhedberg.com/images/3_HM_cDoS2kfNOFzX9pw5SaNP_w=/fit-in/1200x1200/https://s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1856%3A2990%2Fd27ff5b7115fabfcbef77869ec25f768ba538c14.png)

## Key Features

### Auth & Navigation
- **Role Gating**: `requireAgentSession` checks Auth0 for `nickhedberg.com:chat-gpt` roles before rendering any page.
- **App Shell**: A persistent sidebar (Dashboard, Agents, Conversations, Sessions) sits alongside a responsive layout built with CSS modules.
- **Session Awareness**: User info and logout shortcuts live in the sidebar footer, keeping the app aware of the current operator identity.

### Agent Management
- **List View**: `/agents` fetches `GET /ai-agent-api/agents`, displays status badges, fallback warnings, and updated timestamps with `ClientDate`.
- **Detail & Edit**: Dynamic routes (`/agents/[key]`) surface prompt text, tool definitions, unsupported model warnings, and offer edit/delete actions.
- **Proxy Mutations**: All CRUD actions flow through `/ai-agent-admin/api/ai-agent/...`, which injects the master key and streams the upstream response straight back to the browser.

### Conversations & Sessions
- **Conversation Search**: Server-side data fetching pulls paginated conversations with filters so support can jump directly to a customer thread.
- **Session Explorer**: `/sessions` exposes status (`open/closed`), token usage, and overrides pulled from the session store, making it easy to debug a stuck pipeline.
- **Transcript Viewer**: Message timelines render user/assistant/tool entries with structured tool-call payloads for quick triage.

## Technical Architecture

```
mono/ai-agent-admin/
├── pages/
│   ├── api/ai-agent/[...path].js  # Proxy to ai-agent-api
│   ├── agents/                    # List/detail/edit pages
│   ├── conversations/             # Filters + detail views
│   ├── sessions/                  # Session explorer
│   └── dashboard.js               # KPI cards + recent activity
├── components/Layout.js           # Sidebar + shell
├── lib/                           # Auth helpers, API client, env loader
└── styles/                        # CSS modules for shared look/feel
```

- **Proxy Layer**: The API route streams request bodies, preserves headers like `Accept: text/event-stream`, and forwards the caller’s method/path while attaching `Authorization: Bearer AI_AGENT_MK`.
- **Server Components Ready**: Although built on the Pages Router, all data access happens server-side (`getServerSideProps`) to keep secrets off the client.
- **Error UX**: Each page surfaces upstream HTTP status, raw error text, and actionable messaging (403 redirect to `/403`, 401 bounce to Auth0 login).
- **Shared Utilities**: `lib/dateUtils.js` formats timestamps client-side; `lib/auth.js` centralizes login/logout URL construction so all links remain relative to the deployed base path.

AI-Agent-Admin lets me govern dozens of agents and their traffic from a single pane of glass, bringing observability and safe mutation workflows to the whole platform.
