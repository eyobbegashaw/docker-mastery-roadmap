### Core Concept
Containers are lightweight, standalone executable packages including code, runtime, system tools, and dependencies. Unlike VMs, containers share the host kernel but run in isolated user spaces. Each container is an isolated process (or process tree) with its own filesystem, network, and process namespace, but no hardware emulation overhead.

### Key Terminologies
- **Container Image**: Immutable template containing layered filesystem and metadata
- **Container Instance**: Running process from an image with isolated namespaces
- **Container Runtime**: Software executing containers (containerd, CRI-O, runc)
- **Image Registry**: Storage for container images (Docker Hub, ECR, GAR)
- **OCI Specification**: Open standard for image format and runtime behavior
- **Init Process**: PID 1 inside container handling zombie processes and signals

### How it Works
Container creation involves:
1. **Image Pull**: Fetch layers from registry (each layer is tar+gzip with diff metadata)
2. **Rootfs Assembly**: Mount layers via overlayfs to create unified view
3. **Namespace Creation**: Clone new namespaces for PID, NET, MNT, UTS, IPC, USER
4. **cgroup Setup**: Write to `/sys/fs/cgroup/` to set CPU/memory limits
5. **Capability Dropping**: Remove dangerous capabilities (CAP_SYS_ADMIN, CAP_SYS_RAWIO)
6. **Execute**: pivot_root to container rootfs and execve() the entrypoint process

### CLI Commands/Syntax
```bash
# Container lifecycle
docker run -d --name redis --restart unless-stopped redis:alpine
docker pause redis  # Freeze all processes in container
docker unpause redis
docker wait redis  # Block until container stops

# Low-level inspection
docker inspect redis | jq '.[0].State.Pid'
ls -l /proc/$(docker inspect -f '{{.State.Pid}}' redis)/ns/
sudo -E nsenter -t $(docker inspect -f '{{.State.Pid}}' redis) -m -u -i -n -p sh

# Checkpoint/restore (experimental)
docker checkpoint create redis checkpoint1
docker start --checkpoint checkpoint1 redis
```

### Best Practices
- **One process per container** - use supervisord only when absolutely necessary
- **Set health checks** for orchestration: `HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/ || exit 1`
- **Handle signals properly** - ensure PID 1 forwards SIGTERM to child processes
- **Use `--init` flag** to wrap process with tini (reaps zombies, forwards signals)
- **Set restart policies wisely**: `no|on-failure[:max-retries]|always|unless-stopped`

---