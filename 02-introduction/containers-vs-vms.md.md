### Core Concept
VMs virtualize hardware (CPU, memory, disk) via hypervisor, each running full OS. Containers virtualize OS, sharing kernel while isolating processes. This architectural difference creates trade-offs: containers start in milliseconds vs VM minutes, but VMs provide stronger isolation via hardware virtualization extensions (VT-x/AMD-V).

### Key Terminologies
- **Type 1 Hypervisor**: Bare-metal (ESXi, KVM, Hyper-V)
- **Type 2 Hypervisor**: Hosted (VirtualBox, VMware Workstation)
- **Virtual Machine Monitor (VMM)**: Manages VM hardware emulation
- **Paravirtualization**: Guest OS aware of virtualization (virtio drivers)
- **Density**: Number of workloads per physical host
- **Attack Surface**: Code paths accessible to attackers

### How it Works
**VMs**: Hypervisor intercepts privileged instructions and memory accesses, using Extended Page Tables (EPT) for MMU virtualization. Each VM has complete OS kernel, drivers, and libraries. Context switching between VMs requires VMExit/VMEntry operations (thousands of CPU cycles).

**Containers**: Linux kernel implements namespaces for isolation and cgroups for resource limiting. System calls from containers go directly to host kernel (no emulation layer). Process scheduling happens in kernel space without hypervisor overhead (~200 CPU cycles for context switch).

### CLI Commands/Syntax
```bash
# Performance comparison (QEMU/KVM vs Docker)
# VM creation
virt-install --name vm1 --ram 2048 --vcpus 2 --disk size=10 --cdrom ubuntu.iso

# Container creation
docker run -d --cpus=2 --memory=2g ubuntu:22.04 sleep infinity

# Benchmark startup time
time docker run --rm alpine echo "Container ready"
time virsh start vm1 && virsh shutdown vm1

# Resource efficiency check
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
ps aux | grep -E "qemu|kvm"  # VM process overhead
```

### Best Practices
- **Use containers** for microservices, CI/CD runners, dev environments, batch jobs
- **Use VMs** for legacy applications, Windows workloads, strong multi-tenancy requirements
- **Combine both**: Run containers inside VMs for security boundaries (cloud providers do this)
- **Consider kata containers** when needing VM isolation with container workflow
- **Monitor context switch rates** (`vmstat 1`) - high rates indicate noisy neighbor issues

---