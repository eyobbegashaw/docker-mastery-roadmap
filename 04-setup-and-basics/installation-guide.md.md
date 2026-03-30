

 

### Core Concept
Docker installation varies by OS but core components remain: daemon (dockerd), client (docker), containerd, runc. Production deployments require careful storage driver selection, daemon configuration, TLS setup, and integration with system init (systemd). Rootless mode improves security by running daemon as non-root user.

### Key Terminologies
- **Docker Daemon (dockerd)**: Persistent process managing containers, images, networks
- **Docker Socket (/var/run/docker.sock)**: Unix socket for API communication
- **Daemon.json**: Configuration file for dockerd settings
- **Rootless Mode**: Daemon running in user namespace without root privileges
- **Systemd Unit**: Service definition for daemon lifecycle management
- **Live Restore**: Daemon restart without stopping containers

### How it Works
Installation steps:
1. Package manager adds Docker repository and GPG key
2. Installs containerd, runc, docker-ce, docker-ce-cli
3. Systemd unit starts dockerd with config from `/etc/docker/daemon.json`
4. dockerd creates bridge network (docker0) and sets up iptables
5. Adds user to docker group (or configures rootless)

Daemon initialization:
- Parse config (defaults, then daemon.json, then CLI flags)
- Initialize storage driver (auto-detects overlay2)
- Restore existing containers (if live-restore enabled)
- Start API server on Unix socket and optional TCP port
- Initialize graph driver for image storage

### CLI Commands/Syntax
```bash
# Ubuntu/Debian installation
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io

# Rootless installation
curl -fsSL https://get.docker.com/rootless | sh
systemctl --user start docker
export PATH=/home/$USER/bin:$PATH

# Production daemon configuration
cat > /etc/docker/daemon.json <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "live-restore": true,
  "default-ulimits": {
    "nofile": {"Hard": 64000, "Name": "nofile", "Soft": 64000}
  },
  "iptables": true,
  "userland-proxy": false
}
EOF
```

### Best Practices
- **Never expose Docker socket to untrusted containers** (root equivalent)
- **Configure log rotation** aggressively in production (default no rotation)
- **Set `userland-proxy: false`** to use hairpin NAT (better performance)
- **Use separate partition** for `/var/lib/docker` to prevent root filesystem filling
- **Enable live restore** for zero-downtime daemon updates
- **Implement TLS authentication** for remote API access (avoid plain HTTP)
- **Run rootless in multi-tenant environments** (increases security at 5-10% performance cost)

---