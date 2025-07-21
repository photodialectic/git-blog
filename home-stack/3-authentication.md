# Authentication

Authentication is a critical piece of any multi-service architecture. Rather than rolling my own solution, I use Auth0's free tier combined with the NextJS SDK to provide secure, modern authentication across all my HomeStack services.

## Why Auth0?

After evaluating self-hosted options like Keycloak and considering building custom auth, Auth0 emerged as the clear winner:

- **Free tier that scales**: 7,000 monthly active users on the free plan
- **Production-ready security**: OAuth 2.0, OpenID Connect, MFA support
- **Social logins**: Google, GitHub, Microsoft, and more out of the box
- **Zero maintenance**: No security patches, updates, or monitoring needed
- **Developer experience**: Excellent SDKs and documentation

For a HomeStack serving family and friends, the free tier provides plenty of headroom while delivering enterprise-grade security.

## Architecture Overview

My authentication setup uses a centralized auth service that other applications delegate to:

```language-bash
User Request � Traefik � Service � Auth0 SDK � Centralized Auth Service
                                      �
                                  Auth0 Tenant
```

Each service that requires authentication:

1. Redirects unauthenticated users to `/auth/api/login`
2. Receives users back after successful authentication
3. Accesses user information via the Auth0 session

## Centralized Auth Service

The `auth` service handles all OAuth flows and provides authentication endpoints:

```language-javascript
// pages/api/[auth0].js
import { handleAuth, handleLogin, handleLogout } from "@auth0/nextjs-auth0";

export default handleAuth({
  async login(req, res) {
    try {
      await handleLogin(req, res, {
        returnTo: req.query.returnTo || "/",
      });
    } catch (error) {
      res.status(error.status || 500).end(error.message);
    }
  },

  async logout(req, res) {
    try {
      await handleLogout(req, res, {
        returnTo: req.query.returnTo || "/",
      });
    } catch (error) {
      res.status(error.status || 500).end(error.message);
    }
  },
});
```

The `returnTo` parameter allows services to redirect users back to their original destination after authentication.

## Service Integration

Each NextJS service integrates authentication by:

1. **Wrapping the app** with Auth0's UserProvider:

```language-javascript
// _app.js
import { UserProvider } from "@auth0/nextjs-auth0/client";

export default function App({ Component, pageProps }) {
  return (
    <UserProvider
      loginUrl="/auth/api/login"
      profileUrl="/auth/api/me"
      logoutUrl="/auth/api/logout"
    >
      <Component {...pageProps} />
    </UserProvider>
  );
}
```

2. **Adding minimal auth handlers**:

```language-javascript
// pages/api/auth/[...auth0].js
import { handleAuth } from "@auth0/nextjs-auth0";

export default handleAuth();
```

3. **Using the useUser hook** in components:

```language-javascript
import { useUser } from "@auth0/nextjs-auth0/client";

export default function Dashboard() {
  const { user, error, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  if (!user) {
    return <a href="/auth/api/login?returnTo=/chores">Login / Sign Up</a>;
  }

  return <div>Welcome, {user.name}!</div>;
}
```

## Environment Configuration

Services are configured with environment variables pointing to the centralized auth:

```language-yaml
# docker-compose.yml
chat-gpt:
  environment:
    - AUTH0_SECRET=${AUTH0_SECRET}
    - AUTH0_ISSUER_BASE_URL=https://nickhedberg.us.auth0.com
    - AUTH0_CLIENT_ID=DlsKD1TYcN9sNoKPTUDYZ7Ioa6pSYwuM
    - AUTH0_CLIENT_SECRET=${AUTH0_CLIENT_SECRET}
    - AUTH0_BASE_URL=https://www.nickhedberg.com
    - NEXT_PUBLIC_AUTH0_PROFILE=/auth/api/me
```

The same configuration is used across all services, making setup consistent and maintenance simple.

## Multi-Environment Support

I maintain separate Auth0 tenants for development and production:

- **Production**: `nickhedberg.us.auth0.com`
- **Development**: `nickhedberg-dev.us.auth0.com`

This isolation ensures:

- Development testing doesn't affect production users
- Different callback URLs for local vs. deployed services
- Separate user pools and configurations

## User Experience Flow

Here's what the authentication flow looks like for users:

1. **Visit protected service** � Redirected to Auth0 login
2. **Authenticate** � Choose from Google, GitHub, or email/password
3. **Redirected back** � Return to original service with session
4. **Seamless navigation** � Move between services without re-authentication

The single sign-on experience means users authenticate once and access all services in the HomeStack.

## Advanced Features

### Custom User Profiles

Some services extend Auth0's basic user information with application-specific data:

```language-javascript
// After Auth0 login, set up app-specific profile
const setupProfile = async (authUser) => {
  const response = await fetch("/api/user/setup", {
    method: "POST",
    body: JSON.stringify({
      role: "parent", // App-specific fields
      displayName: authUser.name,
    }),
  });
};
```

### Role-Based Access

Services implement their own authorization on top of Auth0 authentication:

```language-javascript
if (!user) {
  return <LoginPrompt />;
}

if (userProfile?.role === "parent") {
  return <ParentDashboard />;
} else if (userProfile?.role === "child") {
  return <ChildDashboard />;
}
```
