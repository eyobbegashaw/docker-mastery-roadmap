 
### Core Concept
Multi-container applications require inter-service communication, data persistence, external dependencies, and often service discovery. Compose simplifies this by creating isolated networks, managing container lifecycles, and handling service dependencies. Real-world apps include web servers, databases, caches, message brokers, and reverse proxies.

### Key Terminologies
- **Service Mesh**: Network of microservices with load balancing and discovery
- **Sidecar Pattern**: Helper container alongside main container (logging, monitoring, proxy)
- **Ambassador Pattern**: Proxy containers abstracting external services
- **Adapter Pattern**: Transform output of one service for another
- **Init Container**: Setup container that runs before main containers

### How it Works
Compose creates dedicated network for each project. Containers discover each other via service names as DNS entries. Example: `app` can connect to `db` hostname (resolves to db container IP). Compose also creates network aliases, links (deprecated), and external connections.

Multi-container orchestration strategies:
1. **Loose coupling**: Services communicate via REST/gRPC
2. **Message queue**: RabbitMQ/Kafka between services
3. **Shared volumes**: For data exchange (avoid when possible)
4. **Database connections**: Connection pooling with PgBouncer sidecar

### CLI Commands/Syntax
```yaml
# LAMP stack with WordPress
version: '3.8'

services:
  wordpress:
    image: wordpress:6-php8.2-apache
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - wp_data:/var/www/html
      - ./custom-theme:/var/www/html/wp-content/themes/custom:ro
    depends_on:
      - db
      - redis
    networks:
      - web

  db:
    image: mariadb:10.11
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - backend

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - wp_data:/var/www/html:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - wordpress
    networks:
      - web

  # Sidecar for log aggregation
  filebeat:
    image: elastic/filebeat:8.10
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - wp_data:/var/log/wordpress:ro
    depends_on:
      - wordpress
    networks:
      - monitoring

networks:
  web:
  backend:
  monitoring:

volumes:
  wp_data:
  db_data:
  redis_data:
```

### Best Practices
- **Use separate networks** for different tiers (web, app, db, cache) for security
- **Implement retry logic** in application code (don't rely solely on depends_on)
- **Use connection pools** for databases to handle container restarts
- **Implement circuit breakers** for external API calls
- **Centralize logging** with ELK/Loki stack (don't rely on `docker logs`)
- **Use `init` containers** for database migrations before app starts
- **Monitor inter-service latency** with distributed tracing (Jaeger, Zipkin)
- **Implement graceful shutdown** (SIGTERM handling) for all services

---