

 

### Core Concept
Dockerfile defines container images through declarative instructions. Each instruction creates a layer, which is cached and reused. Understanding instruction semantics, layer caching, and builder patterns enables creating minimal, secure, efficient images.

### Key Terminologies
- **Build Context**: Files/directories sent to daemon (excluding .dockerignore)
- **Cache Miss**: When instruction changes, invalidates all subsequent layers
- **Multi-stage Build**: Multiple FROM statements copying artifacts between stages
- **Build Arguments (ARG)**: Build-time variables not persisted in final image
- **Environment Variables (ENV)**: Persisted runtime configuration
- **HEALTHCHECK**: Container health probe configuration

### How it Works
Docker build process:
1. Parse Dockerfile, validate syntax
2. Create temporary container for each instruction
3. Execute instruction (e.g., RUN apt-get install)
4. Commit container to new image layer (diff compared to previous)
5. Repeat for each instruction
6. Add metadata (CMD, ENV, EXPOSE, labels)

Layer caching algorithm:
- If instruction has same string, use cached layer
- For COPY/ADD, compare checksums of source files
- For RUN, command string must match exactly (not idempotent)
- `--no-cache` flag forces all layers to rebuild

### CLI Commands/Syntax
```dockerfile
# Multi-stage production Dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o main .

# Stage 2: Runtime
FROM alpine:3.18
RUN apk add --no-cache ca-certificates tzdata
RUN addgroup -g 1001 -S appuser && adduser -u 1001 -S appuser -G appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appuser /app/main .
COPY --chown=appuser:appuser config.yaml ./
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1
ENTRYPOINT ["/app/main"]
CMD ["--config", "/app/config.yaml"]

# .dockerignore (same directory)
*.log
node_modules
.git
Dockerfile
.dockerignore
```

### Best Practices
- **Order layers from least to most frequently changing** (dependencies before code)
- **Use specific tags, not `latest`** in FROM: `FROM python:3.11-slim-bookworm@sha256:...`
- **Combine RUN commands** to reduce layers: `RUN apt-get update && apt-get install -y pkg && apt-get clean && rm -rf /var/lib/apt/lists/*`
- **Use `--link` flag** for COPY (preserves file metadata, better caching)
- **Set `SHELL ["/bin/bash", "-o", "pipefail", "-c"]`** for robust error handling
- **Avoid `apt-get upgrade`** in images (inconsistent base updates)
- **Use `--mount=type=cache` in BuildKit** for persistent package caches:
  ```dockerfile
  RUN --mount=type=cache,target=/var/cache/apt \
      apt-get update && apt-get install -y gcc
  ```

---