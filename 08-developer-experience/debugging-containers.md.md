 

### Core Concept
Debugging containers requires understanding of logging, exec inspection, network debugging, and performance profiling. Production debugging is especially challenging due to ephemeral containers, limited tools, and missing shells in distroless images. Essential techniques include `docker exec`, `nsenter`, sidecar debug containers, and eBPF tools.

### Key Terminologies
- **Debug Container**: Ephemeral container sharing namespaces with target
- **Ephemeral Debugging**: Temporary container with tools for investigation
- **eBPF**: Extended BPF for dynamic tracing without instrumentation
- **Core Dump**: Process memory snapshot for post-mortem analysis
- **Profiling**: CPU/memory sampling to identify bottlenecks
- **Tracing**: Following request flow across services

### How it Works
Debugging layers:
1. **Container level**: Logs, exec, inspect
2. **Host level**: nsenter, /proc, cgroup inspection
3. **Orchestration**: kubectl debug (Kubernetes), ephemeral containers
4. **System level**: eBPF programs hooking kernel events

Common debugging scenarios:
- **High CPU**: `docker exec container top` → `perf top` on host
- **Memory leak**: `docker stats` → `docker exec container cat /proc/meminfo`
- **Network issues**: `nsenter -t PID -n tcpdump` → `netstat -tulpn`
- **Filesystem**: `docker diff container` → `docker cp` files for analysis

### CLI Commands/Syntax
```bash
# Basic debugging
docker logs --tail=100 --timestamps container
docker exec -it container /bin/sh  # Fallback: /bin/bash, sh, or use nsenter
docker inspect container | jq '.[0].State.Health'
docker diff container  # Show changed files

# Network debugging
docker run --rm -it --network container:target nicolaka/netshoot netstat -tulpn
docker run --rm -it --network container:target nicolaka/netshoot tcpdump -i eth0 -c 100
nsenter -t $(docker inspect -f '{{.State.Pid}}' container) -n tcpdump -i eth0

# Performance profiling (requires host access)
docker stats --no-stream container
docker exec container cat /sys/fs/cgroup/cpu.stat
docker exec container cat /proc/$(pgrep node)/limits

# Debugging distroless images (no shell)
docker run --rm -it --pid=container:target alpine sh  # Share PID namespace
nsenter -t $(docker inspect -f '{{.State.Pid}}' target) -m -u -i -n -p

# Core dump collection
docker run --ulimit core=-1 --security-opt seccomp=unconfined app
docker cp container:/core.dump ./
gdb binary core.dump

# eBPF debugging (BCC tools)
sudo /usr/share/bcc/tools/execsnoop  # Trace new processes
sudo /usr/share/bcc/tools/tcpconnect  # Trace outgoing TCP connections
```

### Best Practices
- **Build debug images** with `-debug` tag containing troubleshooting tools (curl, dig, tcpdump, strace)
- **Use ephemeral debug containers** instead of modifying production images
- **Centralize logs** to ELK/Loki for historical debugging
- **Implement distributed tracing** (Jaeger, Zipkin) for microservice debugging
- **Enable core dumps** in staging for crash analysis
- **Use `--init` flag** to prevent zombie process debugging issues
- **Profile regularly** with `docker stats --no-stream` to establish baselines
- **Create debugging playbooks** for common failure scenarios

---