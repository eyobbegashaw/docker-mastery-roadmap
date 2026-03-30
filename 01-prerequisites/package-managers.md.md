## Core Concept

Package managers automate software installation, dependency resolution, and updates. In Docker context, they determine image size, security patching speed, and build reproducibility. Each `RUN apt-get install` creates a layer - understanding this directly impacts optimization.

## Key Terminologies

- **Repository**: Server hosting packages with metadata
- **Dependency Resolution**: Automatic library installation algorithm (SAT solver in APT)
- **Version Pinning**: Locking to specific versions (e.g., `curl=7.68.0`)
- **Cache**: Local metadata storage (`/var/cache/apt/`)
- **GPG Keys**: Cryptographic package signature verification

## How It Works

APT maintains state machine: sources → package lists → dependency graph → download queue → installation. Each package contains control files (maintainer scripts, md5sums) and data.tar.xz archive. DNF uses libsolv with advanced weak dependency support (Recommends/Suggests). Alpine's APK is musl-based, smaller but occasionally incompatible with glibc binaries.

## CLI Commands

```bash
# APT (Debian/Ubuntu) - Most common
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl=7.68.0-1ubuntu2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# DNF (RHEL/Rocky)
RUN dnf install -y --nobest nginx && dnf clean all

# APK (Alpine - smallest images)
RUN apk add --no-cache --virtual .build-deps gcc musl-dev && \
    apk del .build-deps

# Multi-stage build pattern
FROM ubuntu:22.04 AS builder
RUN apt-get update && apt-get install -y build-essential
FROM ubuntu:22.04
COPY --from=builder /usr/local/bin/app /usr/local/bin/app
```

## Best Practices

1. **Combine RUN commands** - Each RUN creates a layer; merge update+install+clean
2. **Use `--no-install-recommends`** (APT) - Reduces image size 30-50%
3. **Pin versions** - `curl=7.68.0` not `curl` (reproducible builds)
4. **Clean caches in same layer** - `rm -rf /var/lib/apt/lists/*`
5. **Use `--no-cache` for APK** - Alpine automatically cleans (no manual rm)
6. **Remove build tools after use** - Install, compile, then uninstall
7. **Prefer official base images** - Ubuntu LTS, Alpine, Red Hat UBI

**Size comparison**: Ubuntu (77MB) → Debian-slim (40MB) → Alpine (5.5MB)