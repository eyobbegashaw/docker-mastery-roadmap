### Core Concept
Open Container Initiative (OCI) standardizes container formats and runtimes, ensuring Docker, containerd, CRI-O, and Podman interoperate. OCI has two specifications: Image Specification (how to store containers) and Runtime Specification (how to run them). Docker implements OCI but adds developer tools (build, push, compose).

### Key Terminologies
- **runc**: OCI reference implementation, CLI for spawning containers
- **containerd**: Industry-standard core container runtime (used by Docker, Kubernetes)
- **CRI-O**: Lightweight OCI runtime for Kubernetes CRI
- **Image Index**: OCI manifest referencing platform-specific images (multi-arch support)
- **Layer Digest**: SHA256 hash ensuring content-addressable storage
- **SELinux/AppArmor**: Mandatory Access Control (MAC) systems for container security

### How it Works
Docker architecture (post-dockershim):
```
Docker CLI → Docker Engine → containerd → runc → container process
```

OCI runtime lifecycle:
1. `runc create` - creates container process paused
2. `runc start` - resumes container execution
3. `runc exec` - attaches new process to existing namespaces
4. `runc kill` - sends signal to container init process

Image pull flow:
Registry (Docker Hub) → containerd pulls manifest → downloads layers by digest → verifies sha256 → unpacks to snapshotter → creates rootfs

### CLI Commands/Syntax
```bash
# Using containerd directly
ctr images pull docker.io/library/nginx:alpine
ctr run --rm --cpus 1 --memory 512M docker.io/library/nginx:alpine nginx-test

# Using runc directly
runc spec  # Generates config.json
runc run mycontainer

# OCI image inspection
skopeo inspect docker://docker.io/library/nginx:latest
crane manifest ubuntu:22.04 | jq '.layers[].digest'

# Multi-architecture builds
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
```

### Best Practices
- **Use containerd** directly for Kubernetes nodes (simpler than Docker Engine)
- **Implement image signing** with cosign or notation for supply chain security
- **Configure registry mirrors** to reduce external bandwidth and improve reliability
- **Set garbage collection policies** for container runtimes to prevent disk exhaustion
- **Understand OCI runtime hooks** for custom network or storage setup pre-start

---