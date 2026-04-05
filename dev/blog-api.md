# Building a Git-Backed Blog API in Go

Git already gives me drafts (branches), history (commits), and content review (pull requests). I wanted an API that exposes that structure directly without inventing another datastore, so I rewrote the blog backend in Go and treated a Git repo as the only source of truth.

## Stack

- [Go](https://go.dev/) 1.23
- [go-git](https://github.com/go-git/go-git) for repo access
- Standard library `net/http`

Everything lives in `mono/blog-api` with a small package layout:

- `cmd/server`: entrypoint
- `internal/config`: env parsing
- `internal/gitrepo`: read-only git client
- `internal/content`: builds filesystem-like responses
- `internal/server`: HTTP handlers + middleware

## API

The API stayed intentionally tiny:

- `GET /health` and `HEAD /health` – readiness checks used by Traefik and DigitalOcean monitors
- `GET /v1/branches` – enumerates branches from `origin`
- `GET /v1/entries` – lists the root tree for the requested branch (defaults to `main`)
- `GET /v1/entries/:path` – drills into any subtree or returns blob contents if the path points to a file

Responses mirror the old service so clients didn’t need to change. The OpenAPI spec (now at `mono/blog-api/docs/openapi.yml`) still documents the same shapes.

## Git workflow

`LOCAL_CONTENT=1` flips the service into “trust the mounted repo” mode, which is perfect for writing locally: bind-mount your checkout to `/content` and the API serves it directly.

When `LOCAL_CONTENT` is unset, the service opens `/content` with go-git and fetches from `origin`. It prefers SSH, but if there’s no agent (common in containers) it automatically retries over HTTPS using `CONTENT_REMOTE_URL` and updates the remote config for the next fetch. Every request walks the HEAD tree, so you’re always seeing exactly what’s in git without extra caching layers.

## Docker targets

The Dockerfile exposes `dev` and `prod` stages:

```dockerfile
FROM golang:1.23 AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod if [ -f go.sum ]; then go mod download; fi
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /out/blog-api ./cmd/server
RUN git clone https://github.com/photodialectic/git-blog.git /content

FROM golang:1.23 AS dev
WORKDIR /app
COPY --from=builder /content /content
COPY . .
ENV PORT=10101
CMD ["go", "run", "./cmd/server"]

FROM gcr.io/distroless/base-debian12:nonroot AS prod
WORKDIR /app
COPY --from=builder /out/blog-api /app/blog-api
COPY --from=builder /content /content
ENV PORT=10101
ENTRYPOINT ["/app/blog-api"]
```

Dev builds keep the toolchain plus source so I can `docker exec` and `go run` inside the container. Prod images stay tiny (distroless + static binary) but still ship the initial `/content` clone from the build stage.

Compose files point to the right target per environment (`prod` for stage/prod, `dev` for dev/local) and local stacks mount the repo + optional `/content` bind.

## Request flow

1. Handler parses `branch` (defaults to `main`).
2. Content service calls `gitrepo.Lookup`, which fetches if needed and walks the tree.
3. If the path maps to a blob, it streams the file contents. Otherwise it builds a nested `{name,type,children}` structure while filtering `.git*` entries.
4. Responses are JSON and mimic a filesystem.

Unit tests cover the content layer (filesystem mode, git mode, blob vs directory) so regressions show up when I `go test ./...`.

## Wrap-up

The API now serves every blog page you’re reading. Draft a post in git, push a branch, hit `/v1/entries?branch=your-draft`, and the content shows up immediately. No migrations, no CMS logins—just git.
