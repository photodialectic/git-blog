# Development and Utilities

A productive HomeStack requires tooling that streamlines common operations and automates repetitive tasks. My development workflow centers around a custom CLI tool and GitHub Actions for continuous deployment, making it easy to manage the complexity of a multi-service monorepo.

## NHDC CLI: HomeStack Command Center

The cornerstone of my development workflow is the NHDC CLI (Nick Hedberg Dot Com CLI) - a Go-based command-line tool that abstracts away the complexity of managing a Docker Compose monorepo.

### Why Build a Custom CLI?

While Make commands provide basic automation, a dedicated CLI offers several advantages:

- **Contextual help**: Built-in documentation for all operations
- **Input validation**: Prevents common mistakes before they happen
- **Template system**: Scaffolds new services with consistent structure
- **Environment awareness**: Seamlessly switches between dev/prod configurations
- **Cross-platform**: Works identically on macOS, Linux, and Windows

### Core Functionality

The CLI wraps common operations with a consistent interface:

```language-bash
# Service lifecycle
nhdc create -t next-js myservice     # Create new service from template
nhdc up myservice                    # Start specific services
nhdc restart myservice              # Restart services
nhdc logs -f myservice              # Follow service logs

# Database operations
nhdc db migrate                      # Run schema migrations
nhdc db shell -u root               # Open MySQL shell

# Secrets management
nhdc secrets encrypt                 # Edit and encrypt secrets
nhdc secrets decrypt                 # Decrypt for local use

# Development utilities
nhdc dev bash myservice             # Interactive shell in container
nhdc dev fmt-js src/                # Format JavaScript code
nhdc dev fmt-py myservice/          # Format Python code
```

### Service Templates

New services are created from templates, ensuring consistency across the monorepo:

- **next-js**: NextJS applications with Auth0 integration
- **tornado**: Python API services with standard structure
- **go**: Go services with health endpoints and proper structure

Templates include:

- Dockerfile with multi-stage builds
- Docker Compose integration with Traefik labels
- Basic application structure and dependencies
- Standard health endpoints and logging
- Authentication integration when requested

### Configuration Management

The CLI uses a YAML configuration file for environment-specific settings:

```language-yaml
# ~/.nhdc/config.yaml
default_env: dev

docker_compose:
  base: "docker-compose.yml"
  dev: "docker-compose-dev.yml"
  prod: "docker-compose-prod.yml"

domains:
  dev: "dev.nickhedberg.com"
  prod: "www.nickhedberg.com"

traefik:
  resolver: "wwwresolver"
```

This allows the CLI to automatically configure services for different environments without manual intervention.

## GitHub Actions CI/CD

Continuous deployment is handled through GitHub Actions, providing automatic deployments when code is pushed to the main branch.

### Deployment Pipeline

The CI pipeline is straightforward but effective:

```language-yaml
# .github/workflows/deploy.yml (conceptual)
name: Deploy to Production
on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        run: |
          ssh ${{ secrets.DEPLOY_USER }}@${{ vars.SERVER_HOST }} << 'EOF'
            cd /path/to/repo
            git pull origin master
            make decrypt_env
            ./scripts/build
          EOF
```

### Deployment Process

When code is pushed to master, the pipeline:

1. **Connects to production server** via SSH using stored GitHub secrets
2. **Pulls latest code** from the repository
3. **Decrypts environment variables** using the age keys on the server
4. **Executes build script** which handles service updates and restarts

This simple but reliable process ensures that production stays in sync with the main branch.

### Security Considerations

The deployment pipeline uses:

- **GitHub secrets** for SSH authentication credentials
- **GitHub variables** for non-sensitive configuration like server hostnames
- **Age encryption** for secrets management on the server
- **SSH key authentication** rather than password-based access

## Development Scripts and Utilities

Beyond the CLI, I maintain a collection of development scripts for common operations:

### Code Formatting

Consistent code formatting is handled through Docker-based formatters:

```language-bash
# JavaScript/TypeScript formatting
./scripts/fmtjs src/
# Uses: docker run --rm -v $(pwd):/work tmknom/prettier --write $1

# Python formatting
./scripts/fmtpy service-name/
# Uses: docker run --rm --volume $(pwd)/$1:/src pyfound/black:latest_release black .
```

These scripts ensure consistent code style without requiring developers to install formatters locally.

### Health Check Management

My HomeStack includes a centralized health monitoring system that provides both human-readable dashboards and machine-readable status for external monitoring.

#### Health Configuration

Services are defined in a YAML configuration file:

```language-yaml
# nickhedberg_site/health.yml
sites:
  - name: AI API
    path: /ai-api/health/liveness
    description: AI API is a LiteLLM Proxy
  - name: Chat GPT
    path: /chat-gpt
    description: Chat UI for OpenAI
  - name: Code Editor
    path: /code-editor
    description: Code editor for JS, HTML, CSS widgets
  # ... more services
```

#### Health Check Handler

The main site includes a health endpoint (`/health`) that asynchronously checks all configured services:

```language-python
class HealthHandler(BaseHandler):
    async def health_check(self, url):
        http_client = httpclient.AsyncHTTPClient()
        try:
            response = await http_client.fetch(url, raise_error=False)
            return response.code
        except Exception:
            return 599

    async def get(self):
        health_config = yaml.safe_load(open("/app/health.yml"))
        sites = health_config["sites"]

        # Build URLs for all services
        host = self.request.headers["x-forwarded-host"]
        proto = self.request.headers["x-forwarded-proto"]
        urls = [f"{proto}://{host}{site['path']}" for site in sites]

        # Make async requests to all services
        reqs = [self.health_check(url) for url in urls]
        responses = await multi(reqs)

        # Aggregate results
        for site, status_code in zip(sites, responses):
            site["status"] = status_code

        # Return worst status as overall status
        self.set_status(max(site["status"] for site in sites))

        # JSON response if requested
        if self.request.headers.get("content-type", "").startswith("application/json"):
            self.write({"sites": sites})
            return

        # HTML dashboard otherwise
        self.render("html/layout.html", page="health", sites=sites)
```

#### Key Features

- **Parallel health checks**: All services are checked concurrently for fast response
- **Fail-fast status**: Overall status reflects the worst individual service status
- **Multiple response formats**: HTML dashboard for humans, JSON for monitoring systems
- **Dynamic routing**: Automatically uses the current domain and protocol
- **External monitoring integration**: DigitalOcean monitoring hits this endpoint

#### DigitalOcean Integration

The health endpoint is registered with DigitalOcean's monitoring service, which:

- **Pings the health endpoint regularly** from multiple geographic locations
- **Alerts on failures** when any service reports non-200 status
- **Provides uptime statistics** and historical monitoring data
- **Integrates with notifications** for immediate failure alerts

### API Documentation Service

My HomeStack includes a dedicated documentation service that provides centralized API documentation for all services through Swagger UI and Redocly interfaces.

#### Documentation Architecture

The docs service is a NextJS application that serves OpenAPI specifications for each service:

```
docs/
├── specs/           # OpenAPI YAML specifications
│   ├── ai-api.yml
│   ├── blog-api.yml
│   ├── chat-gpt.yml
│   ├── chores-api.yml
│   └── ...
├── src/pages/
│   ├── oas/         # Swagger UI routes
│   ├── redoc/       # Redocly UI routes
│   └── api/specs/   # Spec file serving
└── components/      # Navigation and layout
```

#### OpenAPI Specification Management

Each service has its own OpenAPI spec file that defines its API contract:

```language-yaml
# docs/specs/ai-api.yml
openapi: 3.1.0
info:
  title: AI API
  description: LiteLLM Proxy
  version: 1.61.3
servers:
  - url: http://dev.nickhedberg.com/ai-api
    description: Dev
  - url: https://www.nickhedberg.com/ai-api
    description: Prod
security:
  - bearerAuth: []
```

#### Multiple Documentation Interfaces

The docs service provides two different documentation viewers:

**Swagger UI Interface** (`/docs/oas/ai-api.yml`):

```language-javascript
// src/pages/oas/[...slug].js
export default function SwaggerPage() {
  return <SwaggerUI deepLinking={true} url={`/docs/api/specs/${spec}`} />;
}
```

**Redocly Interface** (`/docs/redoc/ai-api.yml`):

- Alternative documentation viewer with different UI/UX
- Better for documentation-heavy APIs
- Enhanced navigation and search capabilities

#### Dynamic Spec Serving

Specification files are served dynamically through an API endpoint:

```language-javascript
// src/pages/api/specs/[...file].js
export default function handler(req, res) {
  const { file } = req.query;
  const filePath = `/app/specs/${file}`;

  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ message: "Not Found" });
  }

  fs.createReadStream(filePath).pipe(res);
}
```

This allows the documentation interfaces to load specifications dynamically without requiring static file hosting.

#### Benefits

This centralized documentation approach provides:

- **Single source of truth** for all API documentation
- **Multiple viewing options** (Swagger UI vs Redocly) based on preference
- **Environment-specific documentation** with correct server URLs
- **Interactive testing** through Swagger UI's "Try it out" feature
- **Version tracking** through Git-managed spec files

The docs service transforms API documentation from an afterthought into a first-class citizen of the HomeStack, making it easy for developers (including future me) to understand and integrate with all available services.
