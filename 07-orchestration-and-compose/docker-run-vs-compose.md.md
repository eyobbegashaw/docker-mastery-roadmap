 

### Core Concept
`docker run` is imperative (how to run), Compose is declarative (what to run). Compose uses YAML to define multi-container applications, handling networks, volumes, dependencies, and environment configuration as code. Compose v2 integrates as Docker CLI plugin with improved performance and compatibility.

### Key Terminologies
- **Compose Specification**: Standardized YAML format (version 3.8+)
- **Project Name**: Namespace for resources (default = directory name)
- **Service**: Container configuration (image, build, ports, volumes)
- **Dependency Management**: `depends_on` and `healthcheck` for startup ordering
- **Profiles**: Conditional service activation
- **Extensions**: `x-` fields for custom metadata or YAML anchors

### How it Works
Compose translates YAML to Docker API calls:
1. Parses `compose.yaml` (or `docker-compose.yml`)
2. Creates network if not exists (`project_default`)
3. Pulls/builds images for each service
4. Creates volumes
5. Creates containers with dependencies (uses `depends_on` condition)
6. Starts containers in dependency order
7. Monitors logs and health status

Compose v2 improvements:
- Written in Go (vs Python for v1)
- Faster startup time (~2x)
- `--watch` for hot reload (beta)
- Direct integration with `docker` CLI

### CLI Commands/Syntax
```yaml
# docker-compose.yml
version: '3.8'

x-logging: &default-logging
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: appdb
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend
    logging: *default-logging

  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
    environment:
      - DB_HOST=db
    volumes:
      - ./config:/app/config:ro
    networks:
      - backend
      - frontend
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - frontend

networks:
  backend:
    driver: bridge
  frontend:
    driver: bridge

volumes:
  db_data:
```

```bash
# Compose commands
docker compose up -d --build --scale app=5
docker compose logs -f --tail=100 app
docker compose exec app sh -c "node --version"
docker compose down -v --rmi local

# Development with hot reload
docker compose watch  # v2.22+ feature

# Profile-based startup
docker compose --profile monitoring up -d
```

### Best Practices
- **Use `depends_on` with `condition: service_healthy`** (not just service start)
- **Define resource limits** for all services to prevent resource exhaustion
- **Use environment variable substitution** with `.env` file for secrets (never hardcode)
- **Implement healthchecks** for all services that expose ports
- **Use profiles** for dev-only services (adminer, mailhog, pgadmin)
- **Pin image digests** in production compose files
- **Separate dev/prod compose files** with `docker-compose.override.yml` for local development
- **Validate compose file**: `docker compose config`

---
