### Core Concept
Docker CLI communicates with daemon via REST API (Unix socket or TCP). Commands follow `docker [context] [command] [subcommand] [flags] [args]` pattern. Understanding CLI efficiency, output formatting, and context switching is crucial for automation and debugging.

### Key Terminologies
- **Docker Context**: Configuration for connecting to multiple daemons
- **BuildKit**: Next-gen builder with parallel stages and cache mounts
- **Docker Compose V2**: `docker compose` plugin (not `docker-compose` legacy)
- **Output Formatting**: Go templates for custom `--format` output
- **Command Aliases**: Custom shortcuts via `~/.docker/config.json`

### How it Works
CLI command flow:
1. Parse flags and arguments
2. Load context from `~/.docker/contexts/`
3. Build API request (HTTP/JSON)
4. Send to daemon via socket (default timeout 60s)
5. Stream response (logs, progress bars, or final output)
6. Render output with optional formatting

BuildKit enhancements:
- Concurrent dependency resolution
- Skip unused stages
- Mount cache directories (`--mount=type=cache`)
- SSH agent forwarding for private repos

### CLI Commands/Syntax
```bash
# Essential management
docker context create prod --docker "host=ssh://user@prod-server"
docker context use prod
docker system prune -a --volumes -f  # Deep clean
docker system events --filter 'type=container' --filter 'event=die'

# Advanced filtering
docker ps --format 'table {{.Names}}\t{{.Status}}' --filter "status=exited" --filter "label=com.example.environment=prod"
docker images --filter "dangling=true" -q | xargs docker rmi

# BuildKit features
export DOCKER_BUILDKIT=1
docker build --secret id=mysecret,src=./secret.txt --ssh default --cache-from=type=registry,ref=myapp:cache .

# Bulk operations
docker rm -f $(docker ps -aq)  # Remove all containers
docker rmi $(docker images -q) -f  # Remove all images

# Monitoring
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker top nginx  # Show processes inside container
```

### Best Practices
- **Use `--quiet` flag** in CI/CD scripts to suppress unnecessary output
- **Set Docker CLI config** for productivity:
  ```json
  {
    "aliases": { "dps": "docker ps --format 'table {{.Names}}\t{{.Status}}'" }
  }
  ```
- **Pipe to `jq` for API debugging**: `docker inspect container | jq '.[0].NetworkSettings.IPAddress'`
- **Use `docker cp` carefully** - consider volume mounts for persistent data
- **Master `--filter` patterns** to avoid complex grep pipelines
- **Understand exit codes**: 0=success, 1=error, 125=daemon error, 126/127=command issues

---