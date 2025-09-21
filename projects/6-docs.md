# [Docs: Interactive API Documentation Platform](/docs)

A NextJS service that provides comprehensive, interactive API documentation for all HomeStack microservices using OpenAPI specifications and modern documentation tooling.

## Overview

As my microservices architecture grew, maintaining accurate and accessible API documentation became crucial. This documentation platform centralizes all API specifications while providing interactive testing capabilities and seamless integration with development workflows.

![Docs Screenshot](https://www.nickhedberg.com/images/D0FOu3DW7-dgTP9ht4yP7NH471g=/fit-in/1200x1200/https://s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1888%3A2850%2F9cba7af0baa83d2518b3374de4a2150c08668434.png)

## Key Features

### Unified Documentation Hub
- **Multi-Service Support**: Single platform for all microservice APIs
- **OpenAPI Integration**: Standards-based documentation using OpenAPI 3.0
- **Interactive Interface**: Live API testing directly from documentation
- **Per-service ownership**: Each service owns its spec in `docs/openapi.yml`, mounted into the Docs service

### Developer Experience
- **Redoc + Swagger UI**: Clean, modern documentation interfaces
- **Code Examples**: Auto-generated code samples in multiple languages
- **Try It Out**: Interactive endpoint testing via a proxy
- **Preview Mode**: Upload a spec in-browser to preview without committing

## Technical Architecture

### NextJS Platform
```
src/
├── pages/
│   ├── api/
│   │   ├── proxy.js               # Live API proxy for Try-It-Out
│   │   └── specs/[...file].js     # Serves mounted specs from /spec
│   ├── oas/[...slug].js           # Swagger UI renderer
│   ├── redoc/[...slug].js         # Redoc renderer
│   └── index.js               # Documentation landing page
└── components/
    ├── Navigation.js          # Service navigation
    └── PageWrapper.js         # Consistent page layout
```

### Specification Management (Mounted per Service)
Each microservice keeps its OpenAPI spec in-repo at `docs/openapi.yml`. The Docs service mounts those files read-only under `/spec` inside the container. Example from `docker-compose.yml`:

```yaml
docs:
  build: ./docs
  volumes:
    - ./ai-api/docs/openapi.yml:/spec/ai-api.yml:ro
    - ./chat-gpt/docs/openapi.yml:/spec/chat-gpt.yml:ro
    - ./chores/docs/openapi.yml:/spec/chores-api.yml:ro
    - ./code-editor/docs/openapi.yml:/spec/code-editor.yml:ro
    - ./mc-skins/docs/openapi.yml:/spec/mc-skins.yml:ro
    - ./tokens/docs/openapi.yml:/spec/tokens.yml:ro
    - ./tools/docs/openapi.yml:/spec/pipeline.yml:ro
    - ./nickhedberg_blog_api/docs/openapi.yml:/spec/blog-api.yml:ro
```

## Implementation Details

### Spec Serving Endpoint
Specs are served from the mounted `/spec` directory using a simple Next API route:

```javascript
// pages/api/specs/[...file].js
const fs = require("fs");

export default function handler(req, res) {
  if (req.method !== "GET") return res.status(405).json({ message: "Method Not Allowed" });
  const { file } = req.query;
  if (!file) return res.status(400).json({ message: "Bad Request" });
  const filePath = `/spec/${file}`;
  if (!fs.existsSync(filePath)) return res.status(404).json({ message: "Not Found" });
  fs.createReadStream(filePath).pipe(res);
}
```

### Interactive Documentation
The platform supports both Redoc and Swagger UI:
- **Responsive Design**: Mobile-friendly documentation interfaces
- **Syntax Highlighting**: Code examples with proper syntax coloring
- **Deep Linking**: Direct links to endpoints and operations
- **Preview Upload**: In-browser spec preview without committing

### API Proxy Integration
```javascript
// Live API testing through proxy
export default async function handler(req, res) {
  const { service, endpoint } = req.query;
  const apiUrl = getServiceUrl(service) + endpoint;
  
  const response = await fetch(apiUrl, {
    method: req.method,
    headers: req.headers,
    body: req.method !== 'GET' ? JSON.stringify(req.body) : undefined
  });
  
  return res.status(response.status).json(await response.json());
}
```
