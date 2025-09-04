
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
- **Version Management**: Support for multiple API versions and deprecation notices

### Developer Experience
- **Redoc Integration**: Clean, modern documentation interface
- **Code Examples**: Auto-generated code samples in multiple languages
- **Try It Out**: Interactive API endpoint testing with real responses
- **Search Functionality**: Full-text search across all API documentation

## Technical Architecture

### NextJS Platform
```
src/
├── pages/
│   ├── api/
│   │   ├── specs.js           # API spec aggregation endpoint
│   │   └── specs/[...file].js # Dynamic spec serving
│   ├── oas/[...slug].js       # OpenAPI Specification viewer
│   ├── redoc/[...slug].js     # Redoc documentation renderer
│   └── index.js               # Documentation landing page
└── components/
    ├── Navigation.js          # Service navigation
    └── PageWrapper.js         # Consistent page layout
```

### Specification Management
```
specs/
├── ai-api.yml          # AI service API specification
├── blog-api.yml        # Blog service API specification  
├── chat-gpt.yml        # Chat application API specification
├── chores-api.yml      # Chores service API specification
├── code-editor.yml     # Code editor API specification
├── mc-skins.yml        # MC Skins service API specification
├── pipeline.yml        # Data pipeline API specification
└── tokens.yml          # Authentication service API specification
```

## Implementation Details

### Dynamic Specification Loading
```javascript
// Automatic spec discovery and serving
const getApiSpecs = () => {
  const specsDir = path.join(process.cwd(), 'specs');
  const specFiles = fs.readdirSync(specsDir)
    .filter(file => file.endsWith('.yml') || file.endsWith('.yaml'))
    .map(file => ({
      name: path.basename(file, path.extname(file)),
      path: `/api/specs/${file}`,
      spec: yaml.load(fs.readFileSync(path.join(specsDir, file)))
    }));
  return specFiles;
};
```

### Interactive Documentation
The platform uses Redoc for rendering interactive documentation:
- **Responsive Design**: Mobile-friendly documentation interface
- **Syntax Highlighting**: Code examples with proper syntax coloring
- **Deep Linking**: Direct links to specific endpoints and operations
- **Export Options**: PDF and HTML export capabilities

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
