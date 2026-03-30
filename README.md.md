# 🐳 Docker Mastery - Complete DevOps Study  

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-20.10%2B-blue)](https://docker.com)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28%2B-blue)](https://kubernetes.io)

**Production-grade Docker learning path based on roadmap.sh with comprehensive technical documentation, real-world examples, and industry best practices.**

## 📚 What You'll Learn

This repository provides a structured, deep-dive curriculum covering:

- **Container Fundamentals** - What containers are, OCI standards, container vs VM
- **Under-the-Hood Architecture** - Namespaces, cgroups, Union Filesystems (OverlayFS)
- **Docker Operations** - Installation, CLI mastery, image building, registry management
- **Data Persistence** - Volumes, bind mounts, backup strategies, storage drivers
- **Orchestration** - Docker Compose, multi-container apps, service dependencies
- **Developer Experience** - Hot reloading, debugging, CI/CD integration
- **Security** - Image scanning, runtime security, seccomp, AppArmor, capabilities
- **Production Deployment** - Kubernetes, cloud PaaS (Cloud Run, ECS, Fly.io)

## 📂 Repository Structure

```
docker-mastery/
├── 01-prerequisites/               # Linux, networking, package managers
├── 02-introduction/                # Container concepts, OCI, VM comparison
├── 03-architecture-under-the-hood/ # Namespaces, cgroups, OverlayFS
├── 04-setup-and-basics/            # Installation, CLI essentials
├── 05-images-and-registries/       # Dockerfiles, optimization, registries
├── 06-data-persistence/            # Volumes, bind mounts
├── 07-orchestration-and-compose/   # Multi-container apps, Compose
├── 08-developer-experience/        # Hot reload, debugging, CI/CD
├── 09-security/                    # Scanning, runtime security
├── 10-deployment-and-beyond/       # K8s, cloud PaaS
└── README.md                       # This file
```

## 🚀 Quick Start

### Prerequisites
- Linux/macOS/Windows with WSL2
- Docker 20.10+ installed
- Basic command-line knowledge

### Clone and Explore
```bash
git clone https://github.com/yourusername/docker-mastery.git
cd docker-mastery

# Start with fundamentals
cat 01-prerequisites/linux-fundamentals.md

# Run examples (requires Docker)
docker run --rm hello-world
```

### Learning Path Recommendation

**Week 1-2**: Prerequisites + Introduction
- Linux fundamentals, networking basics
- Container concepts, Docker architecture

**Week 3-4**: Under the Hood + Setup
- Namespaces, cgroups, filesystems
- Docker installation, CLI commands

**Week 5-6**: Images + Data Persistence
- Writing optimized Dockerfiles
- Volume management, backup strategies

**Week 7-8**: Orchestration + Dev Experience
- Docker Compose for multi-container apps
- Hot reload, debugging, CI/CD pipelines

**Week 9-10**: Security + Production
- Image scanning, runtime security
- Kubernetes basics, cloud deployment

## 🎯 Key Examples

### Multi-Stage Dockerfile (Optimized)
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o main .

FROM alpine:3.18
RUN apk add --no-cache ca-certificates
COPY --from=builder /app/main /main
ENTRYPOINT ["/main"]
```

### Docker Compose (Full Stack)
```yaml
version: '3.8'
services:
  app:
    build: .
    ports: ["8080:8080"]
    depends_on: ["db", "redis"]
  db:
    image: postgres:15-alpine
    volumes: ["db_data:/var/lib/postgresql/data"]
  redis:
    image: redis:7-alpine
volumes:
  db_data:
```

### Security Hardened Run
```bash
docker run --read-only \
  --cap-drop=ALL \
  --cap-add=NET_ADMIN \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  myapp:latest
```

## 📊 Best Practices Summary

| Category | Key Practice |
|----------|---------------|
| **Images** | Multi-stage builds, distroless base, .dockerignore |
| **Security** | Run as non-root, drop capabilities, seccomp profiles |
| **Performance** | Layer caching, buildkit, parallel downloads |
| **Persistence** | Named volumes, regular backups, bind mounts for dev only |
| **Orchestration** | Health checks, resource limits, graceful shutdown |
| **CI/CD** | Cache layers, scan vulnerabilities, promote by SHA |

## 🔧 Tools & Resources

### Essential Tools
- [Dive](https://github.com/wagoodman/dive) - Image layer exploration
- [Trivy](https://github.com/aquasecurity/trivy) - Vulnerability scanner
- [Hadolint](https://github.com/hadolint/hadolint) - Dockerfile linter
- [Docker Bench Security](https://github.com/docker/docker-bench-security) - CIS benchmark
- [Lazydocker](https://github.com/jesseduffield/lazydocker) - Terminal UI

### Learning Resources
- [Official Docker Docs](https://docs.docker.com/)
- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)
- [Play with Docker Classroom](https://training.play-with-docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## 🧪 Testing Your Knowledge

Each module includes:
- **Hands-on exercises** (build, run, debug)
- **Troubleshooting scenarios** (broken containers, network issues)
- **Security challenges** (vulnerable images to fix)
- **Production case studies** (real-world patterns)

### Sample Exercise
```bash
# Challenge: Create a 10MB Go HTTP server container
# Solution: Multi-stage build with scratch base and binary stripping
```

## 🤝 Contributing

Found an error or want to add examples?
1. Fork the repository
2. Create a feature branch
3. Submit a PR with detailed explanation
4. Ensure examples work with latest Docker version

## 📝 License

MIT License - feel free to use for learning, teaching, or production systems.

## ⭐ Show Your Support

If this guide helped you, star the repository and share with fellow DevOps engineers!

---

**Start your Docker mastery journey today!** 🐳

[Report Bug](https://github.com/yourusername/docker-mastery/issues) · [Request Feature](https://github.com/yourusername/docker-mastery/issues)
 
 

 
