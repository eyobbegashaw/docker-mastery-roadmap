### Core Concept
Container networking builds on Linux network namespaces, virtual Ethernet pairs, and bridge devices. Each container gets its own network stack (interfaces, routes, firewall rules). Docker's default bridge network uses NAT to connect containers to external networks, while overlay networks enable multi-host communication via VXLAN tunnels.

### Key Terminologies
- **Network Namespace**: Isolated network stack with own routing table, firewall, and sockets
- **veth pair**: Virtual Ethernet cable - one end in container, other in host bridge
- **Bridge (docker0)**: Layer 2 virtual switch connecting containers
- **NAT (Masquerading)**: Source IP translation for outbound traffic via iptables
- **Port Publishing**: Forward host port to container port (`-p 8080:80`)
- **CNI (Container Network Interface)**: Plugin standard for configuring container networking

### How it Works
When you run `docker run`, Docker creates:
1. Network namespace for container
2. veth pair (vethXXX in host, eth0 in container)
3. Attaches host veth to docker0 bridge
4. Assigns IP from bridge subnet (default 172.17.0.0/16)
5. Sets up iptables rules for port forwarding and NAT

For inter-container communication, packets flow: container eth0 → vethXXX → docker0 bridge → target vethYYY → target container. External access uses DNAT: `iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80`

### CLI Commands/Syntax
```bash
# Bridge network inspection
ip link show docker0
bridge link show  # View veth pairs
iptables -t nat -L -n -v  # Check NAT rules

# Custom network creation
docker network create --driver bridge --subnet=10.5.0.0/16 --ip-range=10.5.0.0/24 --gateway=10.5.0.1 mynet
docker run --network=mynet --ip=10.5.0.50 nginx

# Network troubleshooting
docker run --rm -it --network container:$(docker ps -q -f name=app) nicolaka/netshoot netstat -tulpn
nsenter -t $(docker inspect -f '{{.State.Pid}}' container_name) -n ip addr

# Macvlan for physical network integration
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet
```

### Best Practices
- **Use user-defined bridges** instead of default for automatic DNS resolution between containers
- **Avoid host networking mode** except for performance-critical applications (loses isolation)
- **Implement network policies** with Calico or Cilium for zero-trust container communication
- **Set MTU appropriately** for your infrastructure (default 1500, reduce to 1450 for VXLAN overlays)
- **Monitor conntrack table size** on hosts with high connection churn: `sysctl net.netfilter.nf_conntrack_max`

---