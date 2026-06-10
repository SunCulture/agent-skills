# Golang Best Practices

## Architecture: Adapter Pattern

All Go applications follow the **adapter pattern** (a.k.a. ports-and-adapters / hexagonal architecture).
The core idea: business logic lives in the center with no knowledge of infrastructure. Infrastructure
concerns (HTTP, database, queues, external APIs) are adapters that plug into ports (interfaces) defined
by the domain.

### Project Layout

Use the standard Go project layout adapted for the adapter pattern. This is a guiding structure — not
a rigid template. Adapt directory depth and naming to the complexity of the project:

```
project-root/
├── cmd/
│   └── server/           # or cli/, worker/, etc.
│       └── main.go       # wiring and startup only
├── internal/
│   ├── domain/           # business entities, value objects, domain errors
│   ├── ports/            # interfaces that the domain exposes and expects
│   ├── services/         # use cases / application services (orchestrate domain + ports)
│   ├── adapters/
│   │   ├── http/         # BunRouter handlers, middleware, request/response DTOs
│   │   ├── repository/   # Bun ORM implementations of repository ports
│   │   ├── queue/        # message broker adapters (RabbitMQ, Kafka, NATS, Redis)
│   │   └── external/     # third-party API clients
│   └── config/           # configuration loading, validation
├── migrations/           # Bun SQL migrations
├── pkg/                  # truly reusable library code (use sparingly)
├── Dockerfile
├── docker-compose.yml
├── Makefile
├── go.mod
├── go.sum
└── .env.example
```

### Key Rules

- `cmd/` contains only wiring: parse config → build dependencies → inject into services → start server.
  No business logic here.
- `internal/domain/` has zero imports from `internal/adapters/`. The dependency arrow always points
  inward. Domain types can only depend on the standard library and other domain types.
- `internal/ports/` defines interfaces. Repository ports, service ports, and external service ports
  all live here. Services depend on port interfaces, never on concrete adapters.
- Dependency injection is done manually in `main.go` — no DI frameworks. Go's simplicity makes
  constructor injection straightforward and explicit.

## ORM: Bun

Use [uptrace/bun](https://bun.uptrace.dev/) as the database ORM/query builder.

### Conventions

- Define models in `internal/domain/` or a dedicated `internal/models/` package. Use `bun.BaseModel`
  embedding and struct tags. All primary keys are UUIDv7, generated using
  [google/uuid](https://pkg.go.dev/github.com/google/uuid) v1.6+ (`uuid.Must(uuid.NewV7())`):
  ```go
  import "github.com/google/uuid"

  type User struct {
      bun.BaseModel `bun:"table:users,alias:u"`
      ID        uuid.UUID `bun:"id,pk,type:uuid"`
      Email     string    `bun:"email,notnull,unique"`
      CreatedAt time.Time `bun:"created_at,nullzero,notnull,default:current_timestamp"`
  }

  // In the service/repo layer, generate the ID before insert:
  // user.ID = uuid.Must(uuid.NewV7())
  ```
  Always generate UUIDv7 application-side before persisting. **Do not** add
  `default:gen_random_uuid()` to the column — that produces UUIDv4 (random,
  poor index locality) and silently masks bugs where app-side generation was
  skipped. If you want a database-side safety net, use Postgres 18+'s
  `uuidv7()` function explicitly; otherwise, rely on the constructor pattern
  to enforce ID generation at the domain layer.

- Repository adapters implement port interfaces and accept `*bun.DB` via constructor injection:
  ```go
  type userRepo struct {
      db *bun.DB
  }

  func NewUserRepo(db *bun.DB) ports.UserRepository {
      return &userRepo{db: db}
  }
  ```

- Use Bun's migration system (`migrate` package) for schema changes. Keep migrations in `migrations/`
  with timestamps.

- Prefer Bun's query builder over raw SQL for standard CRUD. Use raw SQL via `bun.NewRaw()` only
  for complex queries where the builder adds unnecessary complexity.

- Always use context propagation: pass `ctx context.Context` to all repository methods.

- Use `bun.Tx` for transactions, obtained via `db.BeginTx(ctx, &sql.TxOptions{})`.

## Routing: BunRouter

Use [uptrace/bunrouter](https://bunrouter.uptrace.dev/) for HTTP routing.

### Conventions

- Define route groups in the HTTP adapter layer, not in `main.go`. `main.go` creates the router
  and passes it to the adapter for route registration:
  ```go
  router := bunrouter.New(
      bunrouter.WithMiddleware(middlewareStack...),
  )
  httpAdapter.RegisterRoutes(router)
  ```

- Handlers receive `bunrouter.Request` and return `error`. Use BunRouter's built-in error handling:
  ```go
  func (h *handler) getUser(w http.ResponseWriter, req bunrouter.Request) error {
      id, err := req.Params().Int64("id")
      if err != nil {
          return bunrouter.NewHTTPError(http.StatusBadRequest, "invalid id")
      }
      // ...
  }
  ```

- Use route groups for versioning and logical grouping:
  ```go
  g := router.NewGroup("/api/v1")
  g.GET("/users/:id", h.getUser)
  g.POST("/users", h.createUser)
  ```

- Middleware follows the standard `func(next bunrouter.HandlerFunc) bunrouter.HandlerFunc` signature.
  Common middleware: request ID, logging, auth, panic recovery, CORS.

## Error Handling

- Always return errors — never `log.Fatal` inside library or adapter code (only acceptable in `main.go`
  during startup).
- Define domain-specific error types in `internal/domain/errors.go`:
  ```go
  var (
      ErrNotFound      = errors.New("not found")
      ErrAlreadyExists = errors.New("already exists")
      ErrUnauthorized  = errors.New("unauthorized")
  )
  ```
- HTTP adapters map domain errors to HTTP status codes. The mapping lives in the HTTP adapter, not
  in the domain.
- Wrap errors with context using `fmt.Errorf("doing X: %w", err)`.

## Testing

- Use the standard `testing` package. Prefer table-driven tests.
- Use [stretchr/testify](https://github.com/stretchr/testify) for assertions (`assert` and `require`).
- Test business logic (services) by mocking port interfaces. Use hand-written mocks or a mock
  generator like `mockgen` or `moq`.
- Test repository adapters against a real database (Postgres in Docker via testcontainers-go or
  a shared test DB). Bun's migration system helps set up test schemas.
- Test HTTP handlers with `httptest.NewRecorder()` and real router dispatch.
- Keep test files next to the code they test: `user_service.go` → `user_service_test.go`.
- Name tests descriptively: `TestUserService_CreateUser_DuplicateEmail_ReturnsAlreadyExists`.

## Observability, Health Probes, and Graceful Shutdown

These are non-negotiable for any Go service. See `references/observability.md`
for the cross-stack contract; the Go-specific patterns (slog, BunRouter
request-ID middleware, `srv.Shutdown(ctx)` on SIGTERM) are documented there.

Every Go service must:
- Use `log/slog` with JSON handler in production.
- Add `RequestIDMiddleware` (UUIDv7 if absent) at the top of the BunRouter chain.
- Initialize OpenTelemetry with `otlptracegrpc` exporter and a sampling policy.
- Expose `/healthz` (no downstream checks) and `/readyz` (DB ping with 1s timeout).
- Listen for `SIGTERM` via `signal.NotifyContext`, flip readiness to fail,
  then call `srv.Shutdown(ctx)` with a 25s deadline.

## Docker

Multi-stage build optimized for Go. **Distroless `nonroot`** runtime —
non-root by default, no shell, ~2 MB base.

```dockerfile
# syntax=docker/dockerfile:1.7
# Build stage
FROM golang:1.23-alpine@sha256:<digest> AS builder
RUN apk add --no-cache git ca-certificates
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/server ./cmd/server

# Runtime stage — distroless, runs as UID 65532 (nonroot)
FROM gcr.io/distroless/static-debian12:nonroot@sha256:<digest>
COPY --from=builder --chown=nonroot:nonroot /app/server /server
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/server"]
```

Resolve the `<digest>` placeholders with `docker pull <image>` followed by
`docker images --digests`. Pin in production manifests — never ship a plain
tag to prod.

`.dockerignore`:

```
.git
.github
**/*_test.go
testdata/
bin/
dist/
*.md
.env
.env.*
!.env.example
```

## Makefile Targets

At minimum, include:

```makefile
.PHONY: build test lint run docker-build docker-up migrate seed clean

build:
	go build -o bin/server ./cmd/server

test:
	go test ./... -v -race -cover

lint:
	golangci-lint run ./...

run:
	go run ./cmd/server

docker-build:
	docker build -t $(APP_NAME) .

docker-up:
	docker compose up -d

migrate:
	go run ./cmd/migrate up

seed:
	go run ./cmd/seed

clean:
	rm -rf bin/ dist/
	go clean -testcache
```
