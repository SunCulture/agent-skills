# Observability Best Practices (Cross-Stack)

Core Principle 8 (see `../SKILL.md`) mandates observability hooks on every backend service:
structured logging, propagated request IDs, metrics endpoints, and health/readiness
probes. This reference is the source of truth for *how* — language-specific references
import these patterns rather than reinvent them.

## What "observability" actually means here

Three signals, each with a clear job:

| Signal | Purpose | Cardinality |
|---|---|---|
| **Logs** | Discrete events with rich context for forensic debugging | High (per-event) |
| **Metrics** | Aggregate numeric series for dashboards, alerts, SLOs | Low (bounded labels) |
| **Traces** | Causal chain across service boundaries for latency attribution | Medium (per-request) |

Plus **probes** (liveness/readiness) and **graceful shutdown** — operational
hygiene that lets the orchestrator do its job.

---

## 1. Structured Logging

### Rules

- **JSON in production**, text-pretty in dev. Never `print()` / `console.log` /
  `fmt.Println` in shipped code.
- **Levels**: `debug`, `info`, `warn`, `error`. Reserve `fatal` for unrecoverable
  startup failures only.
- **Required fields on every log line**: `timestamp` (ISO 8601 / RFC 3339 UTC),
  `level`, `service`, `message`, `request_id` (if in a request scope),
  `trace_id` + `span_id` (if tracing is on).
- **No PII in logs**. Mask emails, tokens, card numbers, full names by default.
  If you need them for debugging, redact at the formatter, not at every call site.
- **Log at boundaries**: inbound request received, outbound call started/finished,
  error caught. Pure logic should rarely log.

### Go (`log/slog`, stdlib since 1.21)

```go
import (
    "log/slog"
    "os"
)

func newLogger(env string) *slog.Logger {
    var handler slog.Handler
    if env == "production" {
        handler = slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelInfo})
    } else {
        handler = slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelDebug})
    }
    return slog.New(handler).With("service", "users-api")
}

// In a handler — attach request-scoped fields
logger := baseLogger.With("request_id", reqID, "user_id", userID)
logger.InfoContext(ctx, "user fetched", "duration_ms", elapsed.Milliseconds())
```

### Python (`structlog`)

```python
import structlog
import logging

def configure_logging(env: str) -> None:
    processors = [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso", utc=True),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
    ]
    if env == "production":
        processors.append(structlog.processors.JSONRenderer())
    else:
        processors.append(structlog.dev.ConsoleRenderer())

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
    )

# In a request — bind context once
structlog.contextvars.bind_contextvars(request_id=req_id, user_id=user_id)
log = structlog.get_logger().bind(service="users-api")
log.info("user fetched", duration_ms=elapsed_ms)
```

### Node.js / TypeScript (`pino`)

```typescript
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  base: { service: "users-api" },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ["req.headers.authorization", "*.password", "*.token"],
  transport: process.env.NODE_ENV === "production"
    ? undefined
    : { target: "pino-pretty" },
});

// Per-request child logger with context
const reqLog = logger.child({ request_id: reqId, user_id: userId });
reqLog.info({ duration_ms: elapsed }, "user fetched");
```

### .NET (`Serilog` with `Microsoft.Extensions.Logging`)

```csharp
// Program.cs
builder.Host.UseSerilog((ctx, services, cfg) => cfg
    .Enrich.FromLogContext()
    .Enrich.WithProperty("service", "users-api")
    .WriteTo.Console(new Serilog.Formatting.Compact.CompactJsonFormatter()));

// In a handler
using (LogContext.PushProperty("request_id", reqId))
using (LogContext.PushProperty("user_id", userId))
{
    logger.LogInformation("user fetched in {DurationMs}ms", elapsedMs);
}
```

---

## 2. Request / Correlation IDs

Every inbound request gets an ID. If the upstream caller already sent one
(`X-Request-Id` or W3C `traceparent`), honor it. Otherwise generate one — UUIDv7
so it's time-sortable.

### Propagation rules

- **Inbound**: middleware reads `X-Request-Id` header; if absent, generates UUIDv7.
- **In-process**: ID lives in request-scoped context (`context.Context` in Go,
  `contextvars` in Python, AsyncLocalStorage in Node, `IHttpContextAccessor` /
  `Activity.Current` in .NET).
- **Outbound HTTP**: client middleware adds `X-Request-Id` and W3C
  `traceparent` headers to every outgoing call.
- **Outbound message queue**: include `request_id` in message headers (AMQP
  properties, Kafka headers, NATS headers, Redis stream fields).
- **Logs**: `request_id` is on every log line in the request scope (see Section 1).

### Go middleware example (BunRouter)

```go
func RequestIDMiddleware(next bunrouter.HandlerFunc) bunrouter.HandlerFunc {
    return func(w http.ResponseWriter, req bunrouter.Request) error {
        id := req.Header.Get("X-Request-Id")
        if id == "" {
            id = uuid.Must(uuid.NewV7()).String()
        }
        ctx := context.WithValue(req.Context(), ctxKeyRequestID, id)
        w.Header().Set("X-Request-Id", id)
        return next(w, req.WithContext(ctx))
    }
}
```

---

## 3. Distributed Tracing (OpenTelemetry)

Use **OpenTelemetry** (OTel) for instrumentation. Vendor-neutral, every major
observability backend ingests OTLP.

### Rules

- **One tracer per service**, named after the service.
- **Auto-instrumentation** for HTTP servers, HTTP clients, DB drivers, message
  brokers — use the language's OTel contrib packages. Don't hand-instrument what
  the libraries already cover.
- **Manual spans** only for meaningful business operations (`charge_card`,
  `enqueue_order`) — not for every function.
- **W3C Trace Context propagation** (`traceparent` + `tracestate` headers) is
  mandatory for HTTP. For queues, propagate trace context in message headers.
- **Sampling**: head-based sampling at the edge (e.g., 10% of traffic, 100% of
  errors). Don't sample-and-pray — set explicit policy.
- **Span attributes**: follow OTel semantic conventions
  (`http.method`, `db.system`, `messaging.system`, etc.). Custom attributes use
  `<service>.<thing>` naming.

### Go bootstrap (sketch)

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    "go.opentelemetry.io/otel/sdk/resource"
    semconv "go.opentelemetry.io/otel/semconv/v1.26.0"
)

func initTracer(ctx context.Context, serviceName, otlpEndpoint string) (func(context.Context) error, error) {
    exporter, err := otlptracegrpc.New(ctx, otlptracegrpc.WithEndpoint(otlpEndpoint), otlptracegrpc.WithInsecure())
    if err != nil { return nil, err }

    res, _ := resource.Merge(resource.Default(),
        resource.NewWithAttributes(semconv.SchemaURL, semconv.ServiceName(serviceName)))

    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(res),
        sdktrace.WithSampler(sdktrace.ParentBased(sdktrace.TraceIDRatioBased(0.1))),
    )
    otel.SetTracerProvider(tp)
    return tp.Shutdown, nil
}
```

### Python (FastAPI + OTel)

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.trace.sampling import ParentBased, TraceIdRatioBased

resource = Resource.create({"service.name": "users-api"})
provider = TracerProvider(resource=resource, sampler=ParentBased(TraceIdRatioBased(0.1)))
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint=otlp_endpoint, insecure=True)))
trace.set_tracer_provider(provider)
FastAPIInstrumentor.instrument_app(app)
```

---

## 4. Metrics

- **Format**: Prometheus exposition (`/metrics` endpoint). OTel can also export
  metrics, but Prometheus scrape is the most portable default.
- **The four golden signals** for every request handler: latency, traffic
  (RPS), errors (rate), saturation (resource utilization). Capture per-route.
- **RED method** (Rate, Errors, Duration) for request-driven services.
  **USE method** (Utilization, Saturation, Errors) for resource-driven
  services (queues, DBs).
- **Histogram buckets**: don't accept defaults. Pick buckets that bracket your
  SLO (e.g., `[0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]` for
  request latency in seconds).
- **Label cardinality**: hard rule — no unbounded labels. Never put `user_id`,
  `email`, or raw URL path with IDs as a label. Use the routed pattern
  (`/users/:id`), not the resolved path.

### Go (`prometheus/client_golang`)

```go
var requestDuration = promauto.NewHistogramVec(
    prometheus.HistogramOpts{
        Name:    "http_request_duration_seconds",
        Help:    "HTTP request latency by route, method, status.",
        Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5},
    },
    []string{"route", "method", "status"},
)
// In middleware:
requestDuration.WithLabelValues(route, req.Method, statusBucket(status)).Observe(elapsed.Seconds())
```

### Python (`prometheus_client`)

```python
from prometheus_client import Histogram, make_asgi_app

REQUEST_DURATION = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["route", "method", "status"],
    buckets=(.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5),
)
app.mount("/metrics", make_asgi_app())
```

---

## 5. Health vs Readiness Probes

Two endpoints, two purposes. Conflating them causes bad outages.

| Endpoint | Question it answers | Failure means |
|---|---|---|
| `/healthz` (liveness) | "Is the process alive and not deadlocked?" | Orchestrator restarts the pod |
| `/readyz` (readiness) | "Can this instance serve traffic right now?" | Orchestrator removes the pod from load balancer rotation, but does NOT restart |

### Rules

- **`/healthz` must NOT check downstream dependencies.** A flaky DB shouldn't
  cause every pod to be killed. Liveness is about the process, not the world.
- **`/readyz` SHOULD check critical downstreams** (DB connectivity, broker
  connectivity) with a short timeout (≤1s). If a dependency is down, this pod
  is not ready to take work — but killing the pod won't help, so liveness stays
  green.
- **On SIGTERM**, flip `/readyz` to fail *immediately*, then drain in-flight
  work, then exit. This lets the load balancer stop sending new requests
  before the pod stops accepting connections.
- **No auth** on these endpoints. They are infrastructure contracts. Bind them
  to the internal network or a separate port if you need access control.

### Go pattern

```go
type Health struct {
    ready atomic.Bool
    db    *bun.DB
}

func (h *Health) Liveness(w http.ResponseWriter, _ *http.Request) {
    w.WriteHeader(http.StatusOK)
    _, _ = w.Write([]byte(`{"status":"ok"}`))
}

func (h *Health) Readiness(w http.ResponseWriter, r *http.Request) {
    if !h.ready.Load() {
        http.Error(w, `{"status":"not_ready"}`, http.StatusServiceUnavailable)
        return
    }
    ctx, cancel := context.WithTimeout(r.Context(), 1*time.Second)
    defer cancel()
    if err := h.db.PingContext(ctx); err != nil {
        http.Error(w, `{"status":"db_unreachable"}`, http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
}
```

### Kubernetes manifest pairing

```yaml
livenessProbe:
  httpGet: { path: /healthz, port: 8080 }
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3
readinessProbe:
  httpGet: { path: /readyz, port: 8080 }
  initialDelaySeconds: 2
  periodSeconds: 5
  failureThreshold: 2
```

---

## 6. Graceful Shutdown

Every service handles SIGTERM. The contract:

1. Receive SIGTERM (or SIGINT in dev).
2. **Flip readiness to fail** so the load balancer stops new traffic.
3. Stop accepting new inbound work (HTTP server stops listening; queue
   consumer stops fetching).
4. Wait for in-flight work to finish, with a deadline (typically 25–30 seconds —
   shorter than the orchestrator's `terminationGracePeriodSeconds`).
5. Close downstream connections (DB pool, broker, tracer flush, log flush).
6. Exit 0.

### Go (HTTP server)

```go
func runServer(ctx context.Context, srv *http.Server, health *Health) error {
    serverErr := make(chan error, 1)
    go func() {
        if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
            serverErr <- err
        }
    }()
    health.ready.Store(true)

    select {
    case <-ctx.Done():
        health.ready.Store(false)
        shutdownCtx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
        defer cancel()
        return srv.Shutdown(shutdownCtx)
    case err := <-serverErr:
        return err
    }
}

// main.go
ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()
if err := runServer(ctx, srv, health); err != nil { ... }
```

### Python (FastAPI lifespan)

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # startup
    db = await create_pool()
    app.state.db = db
    app.state.ready = True
    yield
    # shutdown — runs on SIGTERM via uvicorn
    app.state.ready = False
    await db.close()

app = FastAPI(lifespan=lifespan)
```

Run with `uvicorn --timeout-graceful-shutdown 25` to give in-flight requests
time to finish.

### Node.js (Express)

```typescript
const server = app.listen(PORT);
let isShuttingDown = false;

const shutdown = async (signal: string) => {
  if (isShuttingDown) return;
  isShuttingDown = true;
  logger.info({ signal }, "shutting down");
  app.locals.ready = false;
  server.close(async () => {
    await db.end();
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 25_000).unref();
};
process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT",  () => shutdown("SIGINT"));
```

---

## 7. The non-negotiable observability checklist

A backend service is **not** production-ready unless it has all of:

- [ ] Structured JSON logs in production with `timestamp`, `level`, `service`,
      `message`, `request_id`, `trace_id`.
- [ ] `X-Request-Id` ingress middleware that generates UUIDv7 if absent and
      propagates to outbound calls.
- [ ] OpenTelemetry tracing initialized with a service-name resource attribute
      and a sampling policy (default: 10% + 100% errors).
- [ ] `/metrics` endpoint exposing Prometheus exposition.
- [ ] `/healthz` (liveness, no downstream checks) and `/readyz` (readiness,
      checks critical downstreams with a 1-second timeout).
- [ ] SIGTERM handler that flips readiness to fail, drains in-flight work
      within ≤25s, and closes downstream connections.
- [ ] No PII in logs (formatter-level redaction for tokens, emails, etc.).
- [ ] Histogram buckets explicitly chosen, not framework defaults.

Per-language references reference *this file* for the patterns above.
Don't duplicate; cite.
