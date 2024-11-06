# HomeStack

I thoroughly enjoy creating software. I would classify myself as a full-stack developer, with a preference for back-end development, but I also enjoy tackling front-end challenges.

About 10 years ago, I started working with containers. Before that, I had a more classic approach of developing on provisioned servers or VMs. As soon as I started working with containers, specifically Docker, I knew I wanted my HomeStack to follow a service-oriented architecture.

HomeStack is what I refer to as the tech stack I use for my side projects. I think of it as similar to a mechanic's home garage. It has just enough tools to handle most operations, but it isn't scaled to the level of their day job shop.

Before embracing SOA, I used to host each site or service on a subdomain of my main domain. For example, my web-utils service was on utils.nickhedberg.com. However, with my current HomeStack, everything is on a single domain and the services are differentiated by path using a central router.

In this series, I will write about how I have implemented various infrastructure components in my homestack.

## Posts

[1. Site-Router](/blog/home-stack/1-site-router.md)

The brains of the operation. Originally, I used [https://hub.docker.com/r/jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy), but I have since moved on to fully embracing [Traefik](https://traefik.io/).

[2. Secrets](/blog/home-stack/2-secrets.md)

I leverage [age](https://github.com/FiloSottile/age) for committing encrypted secrets to my git repos.

[3. Authentication](/blog/home-stack/3-authentication.md)

I use the free version of [Auth0](https://auth0.com/) for authentication, along with the [NextJS SDK](https://github.com/auth0/nextjs-auth0).

[4. MySQL](/blog/home-stack/4-mysql.md)

I don't utilize a managed MySQL service; rather, it is another container running on my server. A key unlock for me was managing schema changes, which I use [Skeema](https://www.skeema.io/) for.

[5. APIs and UIs](/blog/home-stack/5-apis-and-uis.md)

For APIs, I mainly use [Tornado](https://www.tornadoweb.org/en/stable/), and for UIs, I use [NextJS](https://nextjs.org/).

[6. Development and Utilities](/blog/home-stack/6-development-and-utilities.md)

I'm a [vim/screen](https://github.com/photodialectic/tilde) user, mainly because I like playing in the terminal still. I do have [CoPilot](https://github.com/github/copilot.vim) installed, and also a modified [vim-ai](https://github.com/photodialectic/vim-ai) plugin for AI Chat.
