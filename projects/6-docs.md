# Docs: Interactive API Documentation Platform

A NextJS service that provides comprehensive, interactive API documentation for all HomeStack microservices using OpenAPI specifications and modern documentation tooling.

## Overview

As my microservices architecture grew, maintaining accurate and accessible API documentation became crucial. This documentation platform centralizes all API specifications while providing interactive testing capabilities and seamless integration with development workflows.

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

### Documentation Management
- **Git-Based Workflow**: Documentation specs stored alongside code
- **Automated Updates**: CI/CD integration for documentation deployment
- **Change Tracking**: Version control for API specification changes
- **Validation**: Automated OpenAPI spec validation and linting

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

## Service Integration

### Automated Spec Collection
Each microservice contributes its API specification:
- **Development Integration**: Specs updated during development cycles
- **CI/CD Pipeline**: Automated spec validation and deployment
- **Version Control**: Specification changes tracked in git history
- **Quality Gates**: Documentation completeness checks in build process

### Cross-Service Documentation
- **Service Discovery**: Automatic detection of new services and endpoints
- **Dependency Mapping**: Visual representation of service interconnections  
- **Authentication Flow**: Unified documentation for auth across services
- **Error Handling**: Consistent error response documentation

## Developer Workflow Integration

### Documentation-Driven Development
- **Spec-First Design**: API specifications written before implementation
- **Mock Server Generation**: Auto-generated mock endpoints for testing
- **Contract Testing**: Validation that implementations match specifications
- **Client Generation**: Auto-generated API clients from specifications

### Quality Assurance
```yaml
# OpenAPI spec validation pipeline
validation:
  spectral:
    rules:
      - no-unresolved-refs
      - operation-operationId-unique
      - info-description
      - operation-description
      - parameter-description
```

## User Interface Design

### Navigation System
- **Service-Based Organization**: Documentation grouped by microservice
- **Search Integration**: Global search across all API documentation
- **Bookmark Support**: Save frequently referenced endpoints
- **Recent Activity**: Track recently viewed documentation sections

### Responsive Documentation
- **Mobile Optimization**: Touch-friendly interface for mobile devices
- **Progressive Enhancement**: Core functionality without JavaScript
- **Accessibility**: Screen reader support and keyboard navigation
- **Performance**: Fast loading with optimized asset delivery

## Monitoring & Analytics

### Documentation Usage
- **Page Views**: Track which APIs are most frequently accessed
- **Search Queries**: Understand developer information-seeking patterns
- **Feedback Collection**: Gather user feedback on documentation quality
- **Performance Metrics**: Monitor documentation load times and availability

### Content Quality Metrics
- **Coverage Analysis**: Ensure all endpoints have complete documentation
- **Freshness Tracking**: Identify outdated documentation sections
- **Example Validation**: Verify that code examples remain functional
- **Link Checking**: Automated validation of internal and external links

## Development Benefits

### Improved Developer Onboarding
- **Single Source of Truth**: All API information in one location
- **Interactive Learning**: Learn APIs through hands-on testing
- **Comprehensive Examples**: Real-world usage patterns and code samples
- **Quick Reference**: Fast lookup of endpoint details and parameters

### Enhanced Team Collaboration
- **Design Reviews**: API specifications as collaboration artifacts
- **Change Communication**: Clear documentation of API changes
- **Cross-Team Alignment**: Shared understanding of service interfaces
- **Historical Context**: Track the evolution of API designs over time

## Future Enhancements

### Advanced Features
- **GraphQL Integration**: Support for GraphQL schema documentation
- **SDK Generation**: Automated client library generation
- **Performance Documentation**: Response time and rate limit information
- **Changelog Automation**: Auto-generated API change notifications

### Developer Tools Integration
- **Postman Collections**: Export collections for API testing
- **Insomnia Support**: Workspace exports for Insomnia users
- **CLI Tools**: Command-line interface for documentation management
- **IDE Plugins**: Integration with popular development environments

The documentation platform serves as the central nervous system for API information across my HomeStack, enabling efficient development and seamless service integration.