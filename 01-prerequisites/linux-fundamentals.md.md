 

### Core Concept
Linux fundamentals form the foundation of Docker operations because containers share the host OS kernel. Understanding Linux process isolation, filesystem hierarchy, and permission models directly impacts container security and performance. Docker essentially packages Linux system calls and kernel features into portable units.

### Key Terminologies
- **Kernel Space**: The privileged part of Linux managing hardware, memory, and processes
- **User Space**: Where user applications run with restricted privileges
- **Process ID (PID)**: Unique identifier for running processes
- **Mount Namespace**: Isolated view of filesystem mount points
- **Capabilities**: Fine-grained privileges broken from root's unlimited power
- **cgroups v2**: Modern resource limitation and prioritization mechanism

### How it Works
Linux uses a monolithic kernel with modular design. When a process makes a system call (e.g., opening a file), it transitions from user mode to kernel mode via software interrupts. Docker leverages this by creating namespaced processes that share the same kernel but have isolated views. The kernel maintains process tables where each entry contains PID, parent PID, user ID, group ID, and namespace references.

### CLI Commands/Syntax
```bash
# Process inspection
ps aux | grep -E "PID|docker"
ls -la /proc/$$/ns/  # View namespaces of current shell

# Permission and capability management
sudo -l  # List allowed sudo commands
getcap /usr/bin/docker  # View Docker binary capabilities
setcap cap_net_raw+ep /usr/bin/custom-binary

# Filesystem hierarchy exploration
df -h  # Disk usage per filesystem
lsblk  # Block device layout
findmnt  # Mounted filesystems with tree structure
```

### Best Practices
- **Never run containers as root** in production; use `--user` flag with non-root UID
- **Drop all capabilities** then add only required ones: `--cap-drop=ALL --cap-add=NET_ADMIN`
- **Use read-only root filesystems** with `--read-only` flag when containers don't write
- **Implement seccomp profiles** to filter system calls - default Docker profile blocks ~44 dangerous syscalls
- **Set resource limits** early: `--cpus=2 --memory=1g --memory-swap=2g` to prevent DoS

---

## 01-prerequisites/package-managers.md

### Core Concept
Package managers automate software installation, updates, and dependency resolution. Docker's layered architecture mirrors package management concepts - each `RUN apt-get install` creates a layer. Understanding APT (Debian/Ubuntu), YUM/DNF (RHEL/CentOS), and apk (Alpine) directly affects Docker image size, security patching, and build reproducibility.

### Key Terminologies
- **Repository**: Server hosting packages and metadata (e.g., packages.docker.com)
- **GPG Keys**: Cryptographic signatures verifying package authenticity
- **Dependency Resolution**: Automatic installation of required libraries
- **Version Pinning**: Locking packages to specific versions (e.g., docker-ce=5:20.10.9)
- **Vendor Directory**: `/etc/apt/sources.list.d/` for third-party repos
- **Cache**: Local package metadata storage (`/var/cache/apt/`)

### How it Works
APT maintains a state machine: sources → package lists → dependency graph → download queue → installation transactions. When running `apt-get install`, APT calculates the minimal set of packages satisfying dependencies using SAT-solving algorithms. Each package contains control files (maintainer scripts, md5sums) and data.tar.xz archive. DNF uses libsolv with a more advanced dependency resolver supporting weak dependencies (Recommends/Suggests).

### CLI Commands/Syntax
```bash
# APT (Debian/Ubuntu)
apt update && apt upgrade -y  # Refresh package lists
apt-cache policy docker-ce  # Show available versions
apt-get install --no-install-recommends -y curl=7.68.0-1ubuntu2

# DNF (RHEL 8+)
dnf check-update
dnf install --nobest -y containerd.io
dnf history undo last  # Rollback transaction

# Alpine APK
apk add --no-cache --virtual .build-deps gcc musl-dev
apk del .build-deps  # Clean build dependencies

# Multi-stage build pattern
FROM ubuntu:22.04 AS builder
RUN apt-get update && apt-get install -y build-essential
FROM ubuntu:22.04
COPY --from=builder /usr/local/bin/app /usr/local/bin/app
```

### Best Practices
- **Combine RUN commands** to avoid layer bloat: `RUN apt-get update && apt-get install -y pkg1 pkg2 && apt-get clean && rm -rf /var/lib/apt/lists/*`
- **Use `--no-install-recommends`** with APT to skip suggested packages (reduces image size 30-50%)
- **Pin base image digests** not tags: `FROM ubuntu@sha256:ac58a67c1f81d79b89f40aa9c8d37d58f6c499e3541cd1c43f225cf93d57b8a2`
- **Implement version manifests** for build reproducibility: `RUN pip install --no-cache-dir -r requirements.txt && pip freeze > /tmp/installed.txt`
- **Clean package manager caches** within same RUN layer to prevent leftover files

---
