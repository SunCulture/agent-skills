# Infrastructure Best Practices

## Docker

### Multi-Stage Builds

Every Dockerfile uses multi-stage builds. The pattern is consistent across stacks:

1. **Builder stage**: install dependencies, compile/build
2. **Runtime stage**: minimal base image, copy only artifacts

Principles:
- Use specific image tags, not `latest` (e.g., `golang:1.23-alpine`, `python:3.12-slim`).
- Place dependency installation before source copy for layer cache efficiency.
- Use `.dockerignore` to exclude `.git`, `node_modules`, `__pycache__`, test files, docs.
- Run as non-root user in the runtime stage:
  ```dockerfile
  RUN addgroup -S app && adduser -S app -G app
  USER app
  ```
- Health checks in the Dockerfile or docker-compose:
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget -qO- http://localhost:8080/health || exit 1
  ```

### Image Size Guidelines

- Go: use `distroless/static` or `alpine` runtime. Final image should be 10-30MB.
- Python: use `python:3.x-slim`. Avoid full Debian images. Final image: 80-150MB.
- Node.js: use `node:22-alpine`. Final image: 100-200MB.
- Always verify final image size with `docker images` and optimize if unexpectedly large.

## Docker Compose / Podman Compose

### Structure

The top-level `version:` key is **obsolete in Compose V2** and emits a
warning. Omit it.

```yaml
# docker-compose.yml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "${APP_PORT:-8080}:8080"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/healthz"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
    networks:
      - backend

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  postgres_data:
  redis_data:

networks:
  backend:
    driver: bridge
```

### Conventions

- Use `depends_on` with `condition: service_healthy` — not just `depends_on` (which only waits
  for container start, not readiness).
- Externalize all config via `.env` file. Provide `.env.example` with placeholder values.
- Named volumes for persistent data. Never bind-mount production data directories.
- Use named networks for service isolation. Don't rely on the default network.
- For podman-compose compatibility: avoid Compose features that podman-compose doesn't support
  (mostly V2+ extensions like `develop`, `watch`). Stick to the V3 spec.
- **Resource limits** (`deploy.resources.limits`) are observed by both
  `docker compose` (V2) and Swarm. Always set them — without limits, a
  runaway container starves its neighbors.

### `.dockerignore` (project root)

Mirror what's in your `.gitignore`, plus build artifacts and secrets:

```
.git
.github
**/.DS_Store
**/node_modules
**/__pycache__
**/.pytest_cache
**/.ruff_cache
**/bin
**/obj
**/dist
**/build
**/.next
**/.nuxt
**/.output
**/coverage
**/*.test.*
**/*.spec.*
.env
.env.*
!.env.example
*.md
LICENSE*
```

A minimal `.dockerignore` per language reference is in `golang.md`,
`python.md`, `frontend.md`, and `csharp.md`.

### Observability and Health Checks

Compose `healthcheck` definitions should hit the application's `/healthz`
endpoint, not `wget` against `/`. See `references/observability.md` for
the contract — `/healthz` is liveness (no downstream checks),
`/readyz` is readiness (with checks, used in K8s/Nomad). Compose only
needs liveness.

For local-dev observability stacks, add an OTLP collector + Grafana / Tempo /
Loki / Prometheus alongside the app container. Wire `OTEL_EXPORTER_OTLP_ENDPOINT`
in `.env`.

## Traefik

### As a Reverse Proxy / API Gateway

```yaml
# docker-compose.yml with Traefik
services:
  traefik:
    image: traefik:v3.2
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.tlschallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL}"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt_data:/letsencrypt
    networks:
      - proxy

  app:
    build: .
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`app.example.com`)"
      - "traefik.http.routers.app.entrypoints=websecure"
      - "traefik.http.routers.app.tls.certresolver=letsencrypt"
      - "traefik.http.services.app.loadbalancer.server.port=8080"
    networks:
      - proxy
      - backend
```

### Key Patterns

- `exposedbydefault=false`: services must explicitly opt-in with `traefik.enable=true`.
- Use Let's Encrypt TLS challenge for automatic SSL. Store certs in a volume.
- Define middleware via labels for rate limiting, auth, headers, circuit breakers:
  ```yaml
  labels:
    - "traefik.http.middlewares.ratelimit.ratelimit.average=100"
    - "traefik.http.middlewares.ratelimit.ratelimit.burst=50"
    - "traefik.http.routers.app.middlewares=ratelimit"
  ```
- For non-Docker setups, use file provider with dynamic configuration in YAML/TOML.

## Nginx

### As a Reverse Proxy

```nginx
# nginx.conf
upstream app_servers {
    server app:8080;
    # Add more backends for load balancing
}

server {
    listen 80;
    server_name app.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name app.example.com;

    ssl_certificate     /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Proxy settings
    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Static files (if applicable)
    location /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Health check endpoint (don't log)
    location /health {
        proxy_pass http://app_servers;
        access_log off;
    }
}
```

### Key Patterns

- Always redirect HTTP to HTTPS in production.
- Set proper proxy headers (`X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`) so the backend
  sees the real client info.
- Use `upstream` blocks for load balancing. Add `least_conn` or `ip_hash` directives as needed.
- Rate limiting with `limit_req_zone` and `limit_req`:
  ```nginx
  limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
  location /api/ {
      limit_req zone=api burst=20 nodelay;
      proxy_pass http://app_servers;
  }
  ```
- Gzip compression for text-based responses:
  ```nginx
  gzip on;
  gzip_types text/plain application/json application/javascript text/css;
  gzip_min_length 1000;
  ```
- Client body size limit: `client_max_body_size 10m;` (adjust per use case).

## .env Management

Every project includes:

- `.env.example` — committed to version control, contains all keys with placeholder/default values.
  Never contains real secrets.
- `.env` — gitignored, contains actual values for local development.
- Document environment variables in README or `.env.example` comments.

```env
# .env.example
APP_PORT=8080
APP_ENV=development

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=changeme

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=changeme

# External Services
API_KEY=your-api-key-here
```
