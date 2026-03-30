### Core Concept
Namespaces provide isolation (what processes see), cgroups limit resources (what processes use). Namespaces wrap global system resources, making processes think they have their own instance. cgroups (control groups) are kernel features limiting, accounting, and prioritizing CPU, memory, disk I/O, and network bandwidth.

### Key Terminologies
- **PID Namespace**: Processes see only themselves and descendants (PID 1 isolation)
- **Mount Namespace**: Filesystem mount points isolation (pivot_root for container root)
- **User Namespace**: UID/GID mapping (host UID 1000000 → container root)
- **Cgroup Namespace**: Virtualized view of cgroup hierarchies
- **Cgroup Subsystems**: cpu, memory, pids, blkio, net_cls, devices
- **CFS (Completely Fair Scheduler)**: CPU bandwidth allocation with quota/period

### How it Works
**Namespace Implementation**:
Each namespace type has system calls: `unshare()` (create new namespace for current process), `setns()` (join existing namespace), `clone()` (create new process in new namespace). The kernel maintains `struct nsproxy` in process descriptor, pointing to namespace structures. PID namespaces add mapping layer: `pid` in container → `virtual pid` → `host pid`.

**cgroups v2 Implementation**:
Hierarchy mounted at `/sys/fs/cgroup/`. Controller interfaces:
- `cpu.max`: "200000 1000000" (20% of 1 CPU)
- `memory.max`: "1073741824" (1GB hard limit)
- `memory.high`: "805306368" (768MB soft limit)
- `pids.max`: "100" (max processes)

### CLI Commands/Syntax
```bash
# Inspect namespaces
lsns  # List all namespaces on system
sudo ls -la /proc/self/ns/  # Current shell namespaces
docker inspect <container> | jq '.[0].State.Pid'

# Create namespaces manually
sudo unshare --fork --pid --mount-proc bash
# Now inside: ps aux shows only this shell

# cgroup manipulation
docker run --cpus=1.5 --memory=2g --memory-swap=4g nginx
cat /sys/fs/cgroup/system.slice/docker-<container-id>.scope/cpu.max

# User namespace remapping (Docker daemon config)
{
  "userns-remap": "default"
}
```

### Best Practices
- **Enable user namespaces** in production for rootless Docker (maps container root to unprivileged host UID)
- **Set `--pids-limit`** to prevent fork bombs: `docker run --pids-limit 100 ...`
- **Monitor pressure stall information (PSI)** for CPU/memory/IO pressure: `/proc/pressure/`
- **Use `--kernel-memory`** limit (deprecated in cgroups v2, use memory.high instead)
- **Configure huge pages** for database containers: `--hugepages-limit=2M:1024`

---
