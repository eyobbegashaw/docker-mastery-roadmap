 

### Core Concept
Bind mounts map host directory directly into container, bypassing Docker storage management. Unlike volumes, bind mounts depend on host filesystem structure and permissions. Essential for development (hot reloading) and when containers need access to host configs (e.g., Docker socket, /proc).

### Key Terminologies
- **Source Path**: Absolute path on host (must exist)
- **Destination Path**: Mount point inside container
- **Mount Propagation**: How mounts propagate between host and container
- **SELinux Labels**: `:Z` (private) and `:z` (shared) for volume labeling
- **Relative Paths**: Not allowed for bind mounts (must be absolute)
- **Subpath Mount**: Mounting subdirectory of a volume or bind mount

### How it Works
Bind mount uses Linux `mount --bind` system call:
1. Kernel creates new mount entry in mount namespace
2. Source and destination inodes are identical (no copy)
3. Changes in either location reflect immediately
4. `stat()` returns same device/inode for both paths
5. Unmounting one doesn't affect other

Permission model:
- Container runs as root (UID 0) → full access to host files
- Container runs as non-root → needs matching host UID/GID
- User namespace remapping can map container UID 1000 → host UID 100000

### CLI Commands/Syntax
```bash
# Development workflow
docker run -v $(pwd):/app -w /app -v /app/node_modules node:18 npm run dev
# Using :delegated (macOS/Windows) for performance
docker run -v $(pwd):/app:delegated node:18 npm test

# Read-only bind mounts
docker run -v /etc/localtime:/etc/localtime:ro alpine date
docker run --mount type=bind,source=/data,target=/data,readonly,bind-propagation=rslave alpine

# SELinux contexts (RHEL/CentOS)
docker run -v /host/data:/container/data:Z  # Private labeling
docker run -v /host/share:/container/data:z  # Shared labeling

# Mount host Docker socket
docker run -v /var/run/docker.sock:/var/run/docker.sock docker:latest docker ps

# Mount individual files
docker run -v ~/.aws/credentials:/root/.aws/credentials:ro amazon/aws-cli s3 ls
```

### Best Practices
- **Never bind mount host root (`/`)** - catastrophic security risk
- **Use `:ro` (read-only) flag** when container doesn't need write access
- **Avoid bind mounts in production** except for config files and logs (use volumes instead)
- **Set `:delegated` or `:cached`** on macOS/Windows for better filesystem performance
- **Use absolute paths** to avoid ambiguity (relative paths not supported)
- **Implement volume drivers for production** (bind mounts tie deployment to host paths)
- **Mount Docker socket cautiously** - gives container root access to host
- **Use `--mount` flag** for complex propagation needs: `type=bind,source=/data,target=/data,bind-propagation=rprivate`

---