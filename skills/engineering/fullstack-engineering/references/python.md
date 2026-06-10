# Python Best Practices

## Architecture: Hexagonal (Ports and Adapters)

All Python applications follow **hexagonal architecture**. The domain core is isolated from
infrastructure. Dependencies point inward — adapters depend on ports, never the reverse.

### Project Layout

```
project-root/
├── src/
│   └── app_name/
│       ├── __init__.py
│       ├── domain/
│       │   ├── __init__.py
│       │   ├── models.py          # entities, value objects, dataclasses
│       │   ├── exceptions.py      # domain-specific exceptions
│       │   └── events.py          # domain events (if event-driven)
│       ├── ports/
│       │   ├── __init__.py
│       │   ├── inbound.py         # use case interfaces (ABCs / Protocols)
│       │   └── outbound.py        # repository & external service interfaces
│       ├── services/
│       │   ├── __init__.py
│       │   └── user_service.py    # use case implementations
│       ├── adapters/
│       │   ├── inbound/
│       │   │   ├── __init__.py
│       │   │   ├── api/           # FastAPI / Flask routes
│       │   │   └── cli/           # CLI entry points
│       │   └── outbound/
│       │       ├── __init__.py
│       │       ├── postgres_repo.py
│       │       ├── redis_cache.py
│       │       └── external_api.py
│       └── config/
│           ├── __init__.py
│           └── settings.py        # pydantic-settings or environ-based config
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── migrations/                    # Alembic or similar
├── Dockerfile
├── docker-compose.yml
├── Makefile
├── pyproject.toml
├── .env.example
└── README.md
```

### Key Rules

- `domain/` has zero imports from `adapters/`. Domain models use only stdlib and domain-level types.
- `ports/` defines abstract interfaces using `typing.Protocol` (preferred) or `abc.ABC`. Services
  depend on these protocols, not on concrete adapter classes.
- Dependency injection: wire in `main.py` or a composition root. Use constructor injection — pass
  adapter instances into service constructors. Avoid DI frameworks unless the project is large enough
  to justify the complexity (in which case `dependency-injector` is acceptable).
- Prefer `dataclasses` or Pydantic `BaseModel` for domain entities. Use Pydantic for DTOs at the
  adapter boundary (request/response schemas).
- **UUIDv7 for all entity IDs**: use the `uuid7` package (`pip install uuid7`) to generate IDs.
  In domain models:
  ```python
  from dataclasses import dataclass, field
  from uuid import UUID
  from uuid_extensions import uuid7

  @dataclass
  class User:
      id: UUID = field(default_factory=uuid7)
      email: str = ""
  ```
  In SQLAlchemy models, store as `UUID` column type (Postgres) or `BINARY(16)` (MySQL).
  Always generate application-side before insert — never rely on database defaults for UUIDv7.

## Framework Preferences

- **REST APIs**: FastAPI (preferred) or Flask. FastAPI for async-first services; Flask for simpler
  sync services.
- **Task queues**: Celery with Redis/RabbitMQ broker, or Dramatiq.
- **ORM**: SQLAlchemy 2.0 (with the new 2.x style) for relational databases. Use the repository
  pattern — SQLAlchemy sessions live in the adapter, never in services.
- **Migrations**: Alembic (with SQLAlchemy) or Django migrations (in Django projects).

## Coding Conventions

- Type hints everywhere. All function signatures, return types, and class attributes should be typed.
  Use `from __future__ import annotations` for forward references.
- Use `pydantic-settings` for configuration, loading from environment variables with `.env` fallback.
- Async where it matters: for I/O-bound services (APIs with DB + HTTP calls), use `async/await`
  throughout. For CPU-bound or simple scripts, sync is fine.
- Logging: use `structlog` or stdlib `logging` with structured output. Never `print()` in production code.
- Linting: code must pass `ruff` checks. Format with `ruff format` or `black`.

## Error Handling

- Define domain exceptions in `domain/exceptions.py`:
  ```python
  class DomainError(Exception):
      """Base for all domain errors."""

  class EntityNotFoundError(DomainError):
      def __init__(self, entity: str, identifier: str):
          super().__init__(f"{entity} not found: {identifier}")
          self.entity = entity
          self.identifier = identifier

  class DuplicateEntityError(DomainError):
      pass
  ```

- Adapters catch infrastructure exceptions and either re-raise as domain exceptions or handle them.
  SQLAlchemy `IntegrityError` → `DuplicateEntityError`, etc.
- FastAPI exception handlers map domain exceptions to HTTP responses. This mapping lives in the
  inbound adapter, not in the domain.
- Never use bare `except:`. Always catch specific exceptions. `except Exception` is acceptable only
  as a top-level catch-all with proper logging.

## Testing

- Use `pytest` as the test runner. Always.
- Structure: `tests/unit/` for isolated logic, `tests/integration/` for adapter tests with real
  infrastructure.
- Fixtures in `conftest.py`. Use `pytest-asyncio` for async tests.
- Mock outbound ports in unit tests using `unittest.mock.AsyncMock` or `MagicMock`.
- Integration tests use `testcontainers` for databases, or a shared Docker Compose test environment.
- Coverage target: meaningful coverage, not vanity metrics. Test business logic thoroughly;
  thin adapter wiring can have lighter coverage.
- Naming: `test_create_user_with_duplicate_email_raises_duplicate_error`.

## Observability, Health Probes, and Graceful Shutdown

Mandatory for every Python backend service. See `references/observability.md`
for the cross-stack contract. Python-specific:

- Configure `structlog` with `JSONRenderer` in production (snippet in
  observability.md).
- Bind `request_id`, `trace_id` to context vars via middleware.
- Use `OpenTelemetry` auto-instrumentation:
  `FastAPIInstrumentor`, `SQLAlchemyInstrumentor`, `HTTPXClientInstrumentor`.
- Expose `/healthz` (always 200) and `/readyz` (DB `SELECT 1` with 1s timeout).
- For FastAPI, use `lifespan` context manager (not deprecated `@app.on_event`)
  for startup/shutdown. Run uvicorn with `--timeout-graceful-shutdown 25`.

## Docker

Multi-stage build for Python. Use **`uv`** (Astral) as the package manager —
10–100× faster than pip/Poetry, deterministic locks, native multi-arch support.
Poetry is acceptable for legacy projects but new projects should use `uv`.

```dockerfile
# syntax=docker/dockerfile:1.7
# Build stage
FROM python:3.12-slim@sha256:<digest> AS builder
COPY --from=ghcr.io/astral-sh/uv:0.5@sha256:<digest> /uv /usr/local/bin/uv
WORKDIR /app
ENV UV_LINK_MODE=copy UV_COMPILE_BYTECODE=1 UV_PYTHON_DOWNLOADS=never
COPY pyproject.toml uv.lock ./
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-dev
COPY src/ ./src/
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Runtime stage — minimal, non-root user
FROM python:3.12-slim@sha256:<digest>
RUN groupadd -g 1001 app && useradd -u 1001 -g app -s /sbin/nologin -M app
WORKDIR /app
COPY --from=builder --chown=app:app /app/.venv /app/.venv
COPY --from=builder --chown=app:app /app/src /app/src
ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1
USER app
EXPOSE 8000
CMD ["uvicorn", "src.app_name.adapters.inbound.api.main:app", \
     "--host", "0.0.0.0", "--port", "8000", \
     "--timeout-graceful-shutdown", "25"]
```

Resolve `<digest>` placeholders with `docker pull <image>` then
`docker images --digests`.

For Poetry-based projects, replace the build stage with:
```dockerfile
RUN pip install --no-cache-dir poetry==1.8.4
COPY pyproject.toml poetry.lock ./
RUN poetry config virtualenvs.in-project true && \
    poetry install --without dev --no-interaction --no-ansi --no-root
```
Note `--without dev` — Poetry's `--no-dev` flag was removed in 1.2.

`.dockerignore`:

```
.git
.github
__pycache__/
**/*.pyc
.pytest_cache/
.ruff_cache/
.mypy_cache/
.venv/
tests/
*.md
.env
.env.*
!.env.example
```

## Makefile Targets

```makefile
.PHONY: install test lint run docker-build docker-up migrate seed clean

install:
	uv sync

test:
	uv run pytest tests/ -v --cov=src --cov-report=term-missing

lint:
	uv run ruff check src/ tests/
	uv run ruff format --check src/ tests/

run:
	uv run uvicorn src.app_name.adapters.inbound.api.main:app --reload

docker-build:
	docker build -t $(APP_NAME) .

docker-up:
	docker compose up -d

migrate:
	uv run alembic upgrade head

seed:
	uv run python -m src.app_name.scripts.seed

clean:
	rm -rf .venv .pytest_cache .ruff_cache .mypy_cache __pycache__
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete
```

For Poetry projects, swap `uv sync` → `poetry install`,
`uv run` → `poetry run`.
