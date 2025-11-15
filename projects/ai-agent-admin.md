# [AI-Agent-Admin: Operator Console](/ai-agent-admin)

A Next.js 15 application that gives operators a secure UI for managing agents, reviewing conversations, and replaying sessions powered by `mono/ai-agent-api`.

## Overview

AI-Agent-Admin is the control room for my conversational stack. It authenticates through Auth0, enforces role-based access, and proxies every request through the same host so the UI never exposes raw API keys. From here I can create/edit agents, audit tool calls, filter conversations by agent, and inspect session state without touching the database.

![AI Agent Admin Screenshot](https://www.nickhedberg.com/images/3_HM_cDoS2kfNOFzX9pw5SaNP_w=/fit-in/1200x1200/https://s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1856%3A2990%2Fd27ff5b7115fabfcbef77869ec25f768ba538c14.png)

## Key Features

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
