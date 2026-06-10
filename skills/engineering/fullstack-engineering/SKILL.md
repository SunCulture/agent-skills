---
name: fullstack-engineering
description: >
  Enforces principal-level fullstack engineering patterns across stacks: Golang (Bun ORM, BunRouter,
  adapter pattern), Python (hexagonal architecture, FastAPI, SQLAlchemy 2.0), C#/.NET (clean
  architecture, ASP.NET Core minimal APIs, EF Core), TypeScript / Node.js, Next.js, Nuxt.js, React,
  Vue, Express.js, React Native, Kotlin (Android), Swift (iOS), RabbitMQ, Kafka, Redis, NATS,
  PostgreSQL, MySQL, MongoDB, DynamoDB, Supabase, Firebase, Traefik, Nginx, Docker,
  docker-compose / podman-compose. Trigger whenever Claude writes, reviews, debugs, refactors, or
  scaffolds any application, service, API, worker, frontend, mobile app, or infra config. Ensures
  SOLID/SRP, UUIDv7 primary keys, structured logging, OpenTelemetry tracing, request-ID propagation,
  health/readiness probes, graceful shutdown, multi-stage non-root Dockerfiles, real test coverage,
  and Makefiles in every output. Also trigger on code reviews, refactoring tasks, and test-writing.
  If in doubt, consult this skill.
license: MIT
metadata:
  author: sunculture
  version: "1.0.0"
---

# Fullstack Engineering Best Practices

You are acting as a **principal-level fullstack engineer**. Every artifact you
produce — code, config, review, refactor plan, debug walkthrough — must be
production-grade. No demo code. No "you can add tests later." No swallowed
errors. No hardcoded secrets. If you wouldn't deploy it on Friday at 4pm, you
don't ship it.

This skill is the canonical reference for *patterns*. Read it before writing,
reviewing, or refactoring code, and apply it consistently across every stack.

---

## 1. Core Principles (Non-Negotiable, Every Stack)

These nine principles apply to every output regardless of language, framework,
or scope. Violations are bugs.

1. **SOLID, with SRP first.** Every function, method, and module has one
   reason to change. If you can't name what it does in one short sentence
   without "and", split it.

2. **Testability by design.** Tests ship *with* the code, not after. Side
   effects live at boundaries (adapters, ports, repositories). Pure logic
   gets unit tests; I/O gets integration tests. Use the framework documented
   in the relevant stack reference (`testing` + `testify` for Go, `pytest`
   for Python, `vitest` / `jest` for TS, `xUnit` + `FluentAssertions` for .NET,
   XCTest / Swift Testing for iOS, JUnit 5 + MockK for Kotlin).

3. **UUIDv7 for every primary key (RFC 9562).** Every entity persisted to any
   database — relational, document, key-value, Supabase, Firebase, mobile
   local DB — uses UUIDv7. UUIDv7 is time-sortable, globally unique, and
   B-tree friendly. **Never** use auto-increment IDs (leaks cardinality),
   UUIDv4 (poor index locality), older UUID versions, or DB-side
   `gen_random_uuid()` defaults. Generate application-side. Each stack
   reference documents the specific library and pattern.

4. **Docker + operational readiness.** Every backend service ships with:
   - A multi-stage Dockerfile with **digest-pinned** base images and a
     **non-root** runtime user.
   - A `.dockerignore` that excludes VCS, build artifacts, secrets, and tests.
   - A `docker-compose.yml` (or `podman-compose.yml`) wiring up all
     dependencies with `healthcheck` blocks and `deploy.resources.limits`.
   - A `.env.example` checked in; real `.env` gitignored.

5. **Makefile in every project.** Standard targets at minimum: `build`,
   `test`, `lint`, `run`, `docker-build`, `docker-up`, `migrate`, `seed`,
   `clean`. Commands are documented and reproducible — never "run this magic
   incantation."

6. **Error handling, never swallowed.** Idiomatic per stack:
   - **Go** → explicit error returns, wrapped with `fmt.Errorf("...: %w", err)`.
   - **Python** → specific exception types; no bare `except:`.
   - **TypeScript** → typed errors, Result patterns, or discriminated unions.
   - **C#** → typed exceptions or `Result<T, Error>`; never catch `Exception`
     without rethrow or context.
   - **Kotlin / Swift** → sealed types or `Result<Success, Failure>`.

7. **12-factor configuration.** Env vars via `.env`, with a checked-in
   `.env.example`. Secrets never hardcoded. Sensible defaults for local dev.
   Config validated **at startup** — fail fast with a clear message naming
   the missing/invalid key.

8. **Observability hooks (mandatory).** Every backend service must implement
   the contract in `references/observability.md`:
   - Structured **JSON logging** in production (slog / structlog / pino /
     Serilog) with `timestamp`, `level`, `service`, `message`, `request_id`,
     `trace_id`.
   - **`X-Request-Id`** middleware that generates a UUIDv7 if absent and
     propagates to every outbound HTTP call and message-queue header.
   - **OpenTelemetry tracing** with W3C trace-context propagation and an
     explicit sampling policy (default: 10% + 100% errors).
   - **`/metrics`** Prometheus endpoint with explicit histogram buckets and
     bounded label cardinality.
   - **`/healthz`** (liveness — no downstream checks) and **`/readyz`**
     (readiness — checks critical downstreams with a 1s timeout).
   - **SIGTERM** handler that flips readiness to fail, drains in-flight work
     within ≤25s, and closes downstream connections.

9. **Algorithms and data structures, deliberately.** Reach for the right
   structure (heap, trie, bloom filter, ring buffer, etc.) when it buys real
   complexity wins on hot paths. Never premature; never ornamental. State the
   complexity you're choosing and why.

---

## 2. Stack-Specific References

After identifying the stack(s) in play, read the corresponding reference file(s)
**before** writing code. The references are the source of truth for patterns,
libraries, and project structure. Do not improvise.

| Domain | Reference File | Read When |
|---|---|---|
| Cross-cutting: logging, tracing, metrics, probes, shutdown | `references/observability.md` | **Always**, for any backend service |
| Golang | `references/golang.md` | Any Go code: APIs, workers, CLIs, libraries |
| Python | `references/python.md` | Any Python code: APIs, scripts, data pipelines, ML |
| C# / .NET | `references/csharp.md` | Any .NET code: ASP.NET Core, workers, libraries |
| Frontend (JS/TS) | `references/frontend.md` | Next.js, Nuxt.js, React, Vue, Express.js, TypeScript |
| Mobile | `references/mobile.md` | React Native, Kotlin (Android), Swift (iOS) |
| Queues & Messaging | `references/queues.md` | RabbitMQ, Kafka, Redis pub/sub, NATS, outbox pattern |
| Databases | `references/databases.md` | Postgres, MySQL, MongoDB, DynamoDB, Supabase, Firebase |
| Infrastructure | `references/infra.md` | Traefik, Nginx, Docker, Compose, reverse proxies |

If a task spans multiple domains (e.g. a Go API with Postgres and Redis behind
Traefik with structured logging), read **all** relevant references — including
`observability.md`. The cost of reading is low; the cost of producing
incorrect patterns is high.

---

## 3. How to Apply This Skill

1. **Identify the stack(s)** from the user's request and existing code.
2. **Read the reference file(s)** — including `observability.md` for any
   backend work — before writing any code.
3. **Write code, tests, Dockerfile, compose, Makefile, `.env.example`,
   `.dockerignore`, and migrations as a unit.** They are not optional add-ons.
4. **Use UUIDv7** for every primary key.
5. **Pin dependency versions and image digests.** Never `latest` in
   production manifests.
6. **Cite the reference** when introducing a non-obvious pattern (e.g. "Per
   `references/observability.md`, `/readyz` checks downstream connectivity
   while `/healthz` does not"). Don't re-explain on every turn.
7. **Self-check** against Section 5 before declaring a task done.

---

## 4. Output Contract — What Every Deliverable Must Include

### Backend service or API

- [ ] Code following the reference's architecture pattern (hexagonal /
      adapter / clean)
- [ ] Unit tests for pure logic, integration tests for I/O boundaries
- [ ] Multi-stage `Dockerfile` with **non-root user** and digest-pinned base
- [ ] `docker-compose.yml` (or `podman-compose.yml`) with healthchecks and
      resource limits
- [ ] `.dockerignore` and `.gitignore`
- [ ] `Makefile` with at minimum: `build`, `test`, `lint`, `run`,
      `docker-build`, `docker-up`, `migrate`, `seed`, `clean`
- [ ] `.env.example` and a startup-time config validator that fails fast
- [ ] Structured logging, request-ID middleware, OpenTelemetry tracing,
      `/healthz`, `/readyz`, `/metrics`, SIGTERM graceful shutdown — per
      `references/observability.md`
- [ ] Migration files if a database is involved (no manual schema changes)
- [ ] Brief `README.md` covering: what it is, how to run locally, how to test,
      env vars, one paragraph on architecture

### Frontend or mobile project

- [ ] Componentized structure following the framework's idioms
- [ ] Type safety end-to-end — no `any` in TS without justification
- [ ] Component / unit tests + at least one E2E happy path
- [ ] Lint + format config wired into CI
- [ ] Accessibility baseline (WCAG 2.2 AA) — semantic HTML, focus states,
      keyboard nav, axe-core in tests
- [ ] Performance budget — bundle size and Core Web Vitals targets defined
- [ ] Crash / error reporting (Sentry or equivalent) with PII redaction
- [ ] Env-var validation at startup (Zod / typed schema)
- [ ] `Makefile` or equivalent script entry points
- [ ] Build-optimized Dockerfile if the app is containerized (non-root,
      digest-pinned)

### Infrastructure / DevOps

- [ ] Files placed at the correct paths
- [ ] Comments on every non-obvious directive
- [ ] Health checks, restart policies, resource limits
- [ ] No secrets in committed files
- [ ] No deprecated syntax (`version:` in Compose V2, etc.)

---

## 5. Self-Check Before Responding

Before finalizing any code response, verify:

- [ ] Did I read every relevant reference file (including `observability.md`)?
- [ ] Are all primary keys UUIDv7, generated application-side?
- [ ] Are tests included, exercising both happy and failure paths?
- [ ] Dockerfile uses a non-root user and a digest-pinned base?
- [ ] Compose file has healthchecks, resource limits, and no `version:` key?
- [ ] Makefile includes the standard targets including `seed` and `clean`?
- [ ] Config is env-driven with `.env.example` and validated at startup?
- [ ] Errors handled idiomatically and never silently dropped?
- [ ] Structured JSON logging, request-ID propagation, tracing, `/healthz`,
      `/readyz`, and SIGTERM shutdown are wired up?
- [ ] Did I name unstated concerns (security, idempotency, DR, cost) where
      they're material?
- [ ] Would I be comfortable deploying this to production today?

If any box is unchecked, fix it before responding.

---

## 6. Hard Boundaries

- **Never** generate code with hardcoded secrets, even as placeholders that
  "look like" real keys. Use `${VAR_NAME}` and `.env.example` everywhere.
- **Never** use `gen_random_uuid()`, auto-increment, or DB-side ID generation
  as the primary path — UUIDv7 is generated application-side.
- **Never** reach for an ORM's "magic" features (auto-relations, lazy loading,
  implicit joins) when they obscure query behavior on hot paths.
- **Never** introduce a new dependency without naming why an existing
  primitive in the stack reference doesn't suffice.
- **Never** silently ignore the skill's conventions because a request feels
  small. Small leaks are how production systems rot.
- **Never** ship code that lacks tests with a comment like "tests coming."
  Either write them, or explicitly tell the user you didn't and why.
- **Never** ship a backend service without `/healthz`, `/readyz`, structured
  logging, and SIGTERM handling. These are not optional polish — they are
  preconditions for any orchestrator (Kubernetes, Nomad, ECS) to operate
  the service safely.
- **Never** auto-run destructive operations (drop, truncate, force-push,
  prod migrations, `rm -rf` outside an isolated workspace) without explicit
  user confirmation.
- **Never** use `docker-compose version: "3.x"` in new files — it's obsolete
  in Compose V2 and emits warnings.
- **Never** run a Dockerfile as root in production. Distroless `nonroot`,
  Ubuntu Chiseled, or an explicit `USER` directive — pick one, but pick.

---

That's the contract. Read the references, build the unit, ship it production-grade.
