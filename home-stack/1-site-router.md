# Site Router

The site router is the heart of my HomeStack infrastructure - it's what makes the single-domain, path-based routing architecture possible. After starting with nginx-proxy, I migrated to Traefik and haven't looked back.

## Why Traefik?

Traefik excels at dynamic service discovery and automatic SSL certificate management. Unlike nginx-proxy which required manual configuration, Traefik automatically discovers services through Docker labels and configures routing rules on the fly.

The key advantages that sold me on Traefik:

- **Automatic service discovery**: No config files to maintain - just Docker labels
- **Built-in Let's Encrypt integration**: SSL certificates are automatically obtained and renewed
- **Path-based routing**: Multiple services can share a single domain with different paths
- **Zero-downtime deployments**: Services can be updated without affecting the router

## Configuration

My Traefik setup uses Docker Compose with configuration passed as command-line arguments. Here's the core `site-router` service:

```yaml
site-router:
  container_name: site-router
  image: traefik:v3.3.1
  command:
    - --providers.docker
    - --providers.docker.network=mono_default
    - --providers.docker.exposedbydefault=false
    - --entryPoints.web.address=:80
    - --entryPoints.web.http.redirections.entrypoint.to=websecure
    - --entrypoints.web.http.redirections.entrypoint.scheme=https
    - --entryPoints.websecure.address=:443
    - --certificatesresolvers.wwwresolver.acme.httpchallenge=true
    - --certificatesresolvers.wwwresolver.acme.httpchallenge.entrypoint=web
    - --certificatesresolvers.wwwresolver.acme.email=nhedberg@me.com
    - --certificatesresolvers.wwwresolver.acme.storage=/letsencrypt/acme.json
  ports:
    - 80:80
    - 443:443
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./letsencrypt:/letsencrypt
```

Key configuration points:

- **Docker provider**: Traefik watches Docker events to discover services
- **Expose by default disabled**: Services must opt-in with labels
- **HTTP to HTTPS redirect**: All traffic automatically upgraded to SSL
- **Let's Encrypt**: Automatic certificate management using HTTP challenge

## Service Integration

Each service in my stack integrates with Traefik using Docker labels. Here's a typical example from my chat-gpt service:

```yaml
chat-gpt:
  labels:
    - traefik.enable=true
    - traefik.http.services.chat-gpt.loadbalancer.server.port=10109
    - traefik.http.routers.chat-gpt.rule=Host(`www.nickhedberg.com`) && PathPrefix(`/chat-gpt`)
    - traefik.http.routers.chat-gpt.entrypoints=websecure
    - traefik.http.routers.chat-gpt.tls.certresolver=wwwresolver
```

This configuration:

- Enables Traefik routing for the service
- Defines the internal port Traefik should proxy to
- Sets up path-based routing (`/chat-gpt` routes to this service)
- Configures HTTPS with automatic certificates

## Advanced Features

### Strip Prefix Middleware

Some services need URL path modification. For example, my blog API expects requests without the `/blog-api` prefix:

```yaml
nickhedberg_blog_api:
  labels:
    - traefik.http.routers.nickhedberg_blog_api.middlewares=nickhedberg_blog_api-stripprefix
    - traefik.http.middlewares.nickhedberg_blog_api-stripprefix.stripprefix.prefixes=/blog-api
```

### Domain Redirects

My main site handles both `nickhedberg.com` and `www.nickhedberg.com`, with automatic redirects:

```yaml
nickhedberg_site:
  labels:
    - traefik.http.routers.nickhedberg_site.rule=Host(`www.nickhedberg.com`) || Host(`nickhedberg.com`)
    - traefik.http.routers.nickhedberg_site.middlewares=addwww-nickhedberg_site
    - traefik.http.middlewares.addwww-nickhedberg_site.redirectregex.regex=^https?://nickhedberg\.com(.*)
    - traefik.http.middlewares.addwww-nickhedberg_site.redirectregex.replacement=https://www.nickhedberg.com$$1
```

## Benefits in Practice

This setup has transformed how I deploy and manage services:

1. **New services are instantly routable** - just add the labels
2. **SSL is completely automated** - certificates appear and renew without intervention
3. **No service downtime** during Traefik updates or configuration changes
4. **Clean URLs** - everything lives under `www.nickhedberg.com/service-name`
5. **Development parity** - same routing logic works locally and in production

The router handles all the networking complexity, letting me focus on building features instead of managing infrastructure. It's the foundation that makes my service-oriented HomeStack architecture practical and maintainable.
