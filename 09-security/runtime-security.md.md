 

### Core Concept
Runtime security protects running containers from exploits, misconfigurations, and malicious behavior. Defense-in-depth includes seccomp (syscall filtering), AppArmor/SELinux (MAC), capabilities dropping, read-only rootfs, and runtime detection (Falco). Zero-trust principle: never trust container even after scanning.

### Key Terminologies
- **Seccomp**: Secure computing mode (syscall whitelist/blacklist)
- **AppArmor**: Mandatory Access Control for paths and capabilities
- **Seccomp Profile**: JSON defining allowed syscalls (default blocks 44 syscalls)
- **Runtime Detection**: Monitoring container behavior anomalies (Falco, Tetragon)
- **Privilege Escalation**: Process gaining higher privileges than intended
- **Immutable Infrastructure**: Containers never modified after deployment

### How it Works
Linux security layers (from least to most restrictive):
1. **Capabilities**: Drop dangerous (CAP_SYS_ADMIN, CAP_SYS_RAWIO)
2. **Seccomp**: Filter syscalls (block `clone` with CLONE_NEWNS)
3. **AppArmor**: Path-based restrictions (deny /etc/passwd write)
4. **SELinux**: Type enforcement (container_t vs svirt_lxc_net_t)

Runtime monitoring with Falco:
- Hooks kernel syscalls via eBPF
- Matches against rules (e.g., "spawn shell in container")
- Sends alerts to SIEM, Slack, or webhook

### CLI Commands/Syntax
```bash
# Runtime security configuration
docker run --security-opt=no-new-privileges \
           --cap-drop=ALL \
           --cap-add=NET_ADMIN \
           --cap-add=NET_RAW \
           --read-only \
           --tmpfs /tmp \
           --security-opt seccomp=/path/to/custom.json \
           --security-opt apparmor=my-profile \
           nginx

# Custom seccomp profile (block unshare)
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["unshare", "clone", "fork"],
      "action": "SCMP_ACT_ERRNO",
      "errnoRet": 1
    }
  ]
}

# Falco runtime detection
helm install falco falcosecurity/falco --set ebpf.enabled=true
# Detects: reverse shells, privileged containers, sensitive mounts

# Audit container syscalls
docker run --security-opt seccomp=unconfined --cap-add=SYS_PTRACE ubuntu \
  strace -c -f -p 1

# Check security configuration
docker inspect --format='{{json .HostConfig.SecurityOpt}}' container
docker run --rm -it --privileged alpine cat /proc/self/status | grep Cap
```

### Best Practices
- **Run containers as non-root** with `--user` flag and matching UID
- **Drop all capabilities** then add only required: `--cap-drop=ALL --cap-add=NET_ADMIN`
- **Enable seccomp profiles** (default is good, customize for your app)
- **Set read-only rootfs** with writable tmpfs: `--read-only --tmpfs /tmp:rw,noexec,nosuid,size=100m`
- **Disable privilege escalation**: `--security-opt=no-new-privileges`
- **Use AppArmor/SELinux** in production environments (RHEL/CoreOS)
- **Implement admission controllers** (Kyverno, OPA) to enforce policies
- **Monitor for anomalous behavior** with Falco or Tetragon
- **Limit resource usage** to prevent DoS: `--cpus=1 --memory=512m`
- **Regularly audit security config** with `docker-bench-security` (CIS benchmark)

---