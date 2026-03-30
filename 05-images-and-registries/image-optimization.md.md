 

### Core Concept
Optimized images reduce attack surface, deployment time, storage costs, and bandwidth. Optimization spans base image selection, layer minimization, dependency management, and final artifact reduction. Alpine Linux is popular but consider distroless images for maximum security.

### Key Terminologies
- **Distroless Images**: Minimal images without shell or package manager (Google Distroless)
- **Slim Images**: Official images with `-slim` suffix (Debian-based, stripped)
- **Scratch Image**: Empty image for statically compiled binaries
- **Layer Squashing**: Merging all layers into single layer (reduces size but loses caching)
- **Deduplication**: Shared layers between images via content-addressable storage
- **Compression Algorithms**: gzip (default), zstd (better ratio/speed)

### How it Works
Size reduction techniques:
1. **Base image size**: Ubuntu (77MB) → Alpine (5.5MB) → Distroless (2MB) → Scratch (0MB)
2. **Layer combining**: Multiple RUN commands create intermediate data (pip cache, temp files)
3. **Binary stripping**: Remove debug symbols (up to 60% size reduction for compiled languages)
4. **Multi-stage builds**: Build tools not needed in final image
5. **Deduplication**: `docker save` shows each layer stored once across images

BuildKit optimizations:
- `--mount=type=cache` preserves apt/yarn/pip cache across builds
- `--mount=type=tmpfs` for temporary files that don't persist
- Parallel layer downloads during `docker pull`

### CLI Commands/Syntax
```bash
# Image analysis
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}"
docker system df -v  # See each layer size
docker history --no-trunc myimage:latest

# Layer inspection
dive myimage:latest  # Interactive layer exploration
docker run --rm -it myimage:latest /bin/sh -c "du -sh /* 2>/dev/null | sort -h"

# Size optimization example
FROM python:3.11-slim AS builder
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH
COPY . .
# Result: 400MB → 120MB

# Export and recompress with better algorithm
docker save myimage:latest | pigz -9 > myimage.tar.gz
```

### Best Practices
- **Start with official `-slim` variants**, switch to Alpine only if compatible (musl vs glibc issues)
- **Use `.dockerignore`** to exclude .git, node_modules, __pycache__, *.log
- **Clean package managers within same RUN**:
  ```dockerfile
  RUN apt-get update && \
      apt-get install -y --no-install-recommends build-essential && \
      make install && \
      apt-get remove -y build-essential && \
      apt-get autoremove -y && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/*
  ```
- **Prefer `COPY --link`** for shared layer deduplication across builds
- **Implement weekly base image updates** to receive security patches (use Dependabot/Renovate)
- **Use `--chown` in COPY** to avoid `RUN chown ...` layers
- **Consider `upx` compression** for binaries (trade-off: startup decompression time)

---