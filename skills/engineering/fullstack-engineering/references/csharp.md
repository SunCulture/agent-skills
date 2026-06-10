# C# / .NET Best Practices

Targets **.NET 9** (current LTS as of writing). Patterns also work on .NET 8.
Use `dotnet --version` to confirm SDK; pin via `global.json`.

## Architecture: Clean / Hexagonal

Same shape as Go and Python in this skill — domain core has no infrastructure
imports; adapters depend inward on domain ports.

### Project Layout

```
solution-root/
├── src/
│   ├── App.Domain/                   # entities, value objects, domain errors
│   │   ├── Users/
│   │   │   ├── User.cs
│   │   │   └── UserErrors.cs
│   │   └── Common/
│   │       └── Result.cs
│   ├── App.Application/              # use cases / services + port interfaces
│   │   ├── Abstractions/             # IUserRepository, IClock, IEmailSender
│   │   ├── Users/
│   │   │   ├── CreateUser/
│   │   │   │   ├── CreateUserCommand.cs
│   │   │   │   └── CreateUserHandler.cs
│   │   │   └── GetUser/
│   │   └── DependencyInjection.cs
│   ├── App.Infrastructure/           # adapter implementations
│   │   ├── Persistence/              # EF Core DbContext, migrations, repos
│   │   ├── Messaging/                # queue adapters
│   │   ├── External/                 # third-party API clients
│   │   └── DependencyInjection.cs
│   └── App.Api/                      # ASP.NET Core minimal API host
│       ├── Endpoints/
│       ├── Middleware/
│       ├── Program.cs                # composition root
│       └── appsettings.json
├── tests/
│   ├── App.Domain.UnitTests/
│   ├── App.Application.UnitTests/
│   └── App.Api.IntegrationTests/
├── Dockerfile
├── docker-compose.yml
├── Makefile
├── App.sln
├── global.json
├── Directory.Packages.props          # central package management
├── .dockerignore
└── .env.example
```

### Key Rules

- `App.Domain` references **nothing** from this solution and only the BCL.
- `App.Application` references `App.Domain` only.
- `App.Infrastructure` and `App.Api` reference `App.Application`. Composition
  happens in `Program.cs`.
- **No service locators** (`IServiceProvider.GetService` inside business code).
  Constructor inject everything.
- Use `Directory.Packages.props` for **Central Package Management** so versions
  live in one place across the solution.

## ASP.NET Core: Minimal APIs

For new HTTP services, use **minimal APIs** — less ceremony, full feature parity
with controllers. Use controllers only when MVC views are involved.

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services
    .AddApplication(builder.Configuration)
    .AddInfrastructure(builder.Configuration);

builder.Host.UseSerilog((ctx, services, cfg) => cfg
    .Enrich.FromLogContext()
    .Enrich.WithProperty("service", "users-api")
    .WriteTo.Console(new Serilog.Formatting.Compact.CompactJsonFormatter()));

var app = builder.Build();
app.UseRequestId();          // custom middleware, see observability.md
app.UseExceptionHandler();   // ProblemDetails handler
app.MapHealthEndpoints();    // /healthz + /readyz
app.MapPrometheusScrapingEndpoint();

app.MapUserEndpoints();      // extension method that defines /users routes
await app.RunAsync();
```

### Endpoint Conventions

- Group endpoints by aggregate: `MapGroup("/users")` → `MapPost("/")`,
  `MapGet("/{id}")`, etc. Define the group in an extension method per feature.
- Bind via `[FromBody]`, `[FromRoute]`, `[FromQuery]` parameter attributes.
- Validate request DTOs with **FluentValidation** at the boundary; never
  trust input.
- Return typed results: `Results.Ok(user)`, `Results.Created($"/users/{id}", user)`,
  `Results.NotFound()`, `TypedResults.Problem(...)`. Avoid `IActionResult` from
  minimal APIs.
- Map domain errors to HTTP via **RFC 7807 ProblemDetails** in the exception
  handler. Never leak stack traces.

## Persistence: EF Core 9

Default ORM. Use **Dapper** alongside for hot-path read queries where EF's
change tracking adds cost.

### Conventions

- DbContext lives in `App.Infrastructure.Persistence`. Inject via
  `services.AddDbContext<AppDbContext>(...)`.
- One `IEntityTypeConfiguration<T>` per aggregate, in
  `Persistence/Configurations/`. Apply via
  `modelBuilder.ApplyConfigurationsFromAssembly(...)`.
- **UUIDv7 for all primary keys.** .NET 9 ships `Guid.CreateVersion7()` natively
  — use it directly. On .NET 8 or earlier, use the `UUIDNext` NuGet package
  (`Uuid.NewSequential()`).
  ```csharp
  // .NET 9+
  public sealed class User
  {
      public Guid Id { get; init; } = Guid.CreateVersion7();
      public string Email { get; init; } = string.Empty;
      public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
  }
  ```
  Generate the ID **in the domain or application layer**, not in the database.
  Postgres maps `Guid` → `uuid` natively. SQL Server stores as `uniqueidentifier`
  but its byte ordering is wrong for B-tree locality — use `UUIDNext`'s
  `SqlServer` variant when the target is SQL Server.
- Repositories return **domain types**, not EF entities. If your domain entity
  is also your EF entity (acceptable for small services), that's fine — but
  mapping is the contract, not an accident.
- **Migrations**: `dotnet ef migrations add <Name>` from the API project.
  Apply via `dotnet ef database update` in CI/CD, not at app startup.
- Use `AsNoTracking()` for read-only queries. Use `AsSplitQuery()` for
  multi-collection includes. Profile before optimizing further.

## Background Work

- **Hosted Services** (`IHostedService` / `BackgroundService`) for in-process
  workers.
- **Channels** (`System.Threading.Channels`) for in-process producer/consumer.
- For cross-process work, use a real broker per `references/queues.md`.
  MassTransit is acceptable as the abstraction layer over RabbitMQ / Azure
  Service Bus / SNS-SQS — but pick one transport and don't pretend it's
  swappable.

## Configuration

- `IOptions<T>` with `IOptionsSnapshot<T>` (per-request) or `IOptionsMonitor<T>`
  (push updates). Bind sections from `appsettings.json` + environment.
- **Validate at startup** with `services.AddOptions<T>().ValidateDataAnnotations()
  .ValidateOnStart()`. Fail fast.
- Secrets: never in `appsettings.json` committed to git. Use
  `dotnet user-secrets` locally and the platform secret store in production
  (Azure Key Vault, AWS Secrets Manager, GCP Secret Manager).
- **12-factor**: `ENV_VAR_NAME` overrides any config key via `__` separator
  (`"Database__ConnectionString"` → `Database__ConnectionString` env var).

## Error Handling

- Define **domain errors** as discriminated unions or a `Result<T, Error>` type.
  Don't throw for expected business failures (`UserNotFound`, `EmailTaken`).
  Throw for genuinely exceptional infra failures.
  ```csharp
  public abstract record DomainError(string Code, string Message);
  public sealed record UserNotFound(Guid Id)
      : DomainError("user.not_found", $"User {Id} not found");
  public sealed record EmailAlreadyTaken(string Email)
      : DomainError("user.email_taken", $"Email {Email} already exists");
  ```
- A **global exception handler** middleware converts uncaught exceptions to
  RFC 7807 ProblemDetails. Map `DbUpdateException` (unique-violation) →
  409 Conflict; `OperationCanceledException` → 499 Client Closed; everything
  else → 500.
- **Never `catch (Exception)`** without logging and rethrowing or returning a
  meaningful Result. Bare swallow = silent prod incident.

## Testing

- **xUnit** as the test runner. **FluentAssertions** for readable assertions.
  **NSubstitute** (preferred) or **Moq** for mocking — pick one per repo.
- **Unit tests** mock port interfaces, exercise application handlers without
  infrastructure. Coverage focus: business rules, edge cases, error paths.
- **Integration tests** use `WebApplicationFactory<Program>` to host the API
  in-memory and `Testcontainers` (`Testcontainers.PostgreSql`,
  `Testcontainers.RabbitMq`) for real dependencies. Mocks lie about the
  database; testcontainers don't.
- Test naming: `MethodName_Scenario_ExpectedOutcome`
  (`CreateUser_WhenEmailExists_ReturnsEmailTakenError`).
- Use `[Theory]` + `[InlineData]` or `[MemberData]` for parametric tests.
- Coverage tools: `coverlet.collector` for runner-agnostic line coverage.

```csharp
public sealed class CreateUserHandlerTests
{
    private readonly IUserRepository _repo = Substitute.For<IUserRepository>();
    private readonly CreateUserHandler _sut;

    public CreateUserHandlerTests() => _sut = new CreateUserHandler(_repo);

    [Fact]
    public async Task Handle_WhenEmailExists_ReturnsEmailTakenError()
    {
        _repo.ExistsByEmailAsync("a@b.c", Arg.Any<CancellationToken>()).Returns(true);
        var result = await _sut.Handle(new CreateUserCommand("a@b.c", "name"), CancellationToken.None);
        result.Error.Should().BeOfType<EmailAlreadyTaken>();
    }
}
```

## Logging, Tracing, Metrics, Health, Shutdown

See `references/observability.md` — **mandatory reading**. The .NET-specific
hooks:

- **Logging**: Serilog with `CompactJsonFormatter` in production. Bind
  `request_id` and `trace_id` via `LogContext.PushProperty` per request.
- **Tracing**: `OpenTelemetry.Extensions.Hosting` +
  `OpenTelemetry.Instrumentation.AspNetCore` +
  `OpenTelemetry.Instrumentation.EntityFrameworkCore` +
  `OpenTelemetry.Exporter.OpenTelemetryProtocol`. Instruments most things
  for free.
- **Metrics**: `OpenTelemetry.Instrumentation.AspNetCore` exports HTTP
  metrics. Expose Prometheus via `OpenTelemetry.Exporter.Prometheus.AspNetCore`
  (`MapPrometheusScrapingEndpoint()`).
- **Health**: `Microsoft.Extensions.Diagnostics.HealthChecks`. Register
  individual checks (`AddDbContextCheck<AppDbContext>()`,
  `AddRabbitMQ(...)`) and split:
  ```csharp
  app.MapHealthChecks("/healthz", new HealthCheckOptions {
      Predicate = _ => false   // liveness: no checks
  });
  app.MapHealthChecks("/readyz", new HealthCheckOptions {
      Predicate = c => c.Tags.Contains("ready")
  });
  ```
- **Graceful shutdown**: `IHostApplicationLifetime.ApplicationStopping`. ASP.NET
  Core handles HTTP drain automatically; you handle queue consumers and
  background services explicitly.

## Dockerfile (multi-stage, non-root, chiseled runtime)

Use Microsoft's **Ubuntu Chiseled** runtime (`-jammy-chiseled`) — minimal,
runs as `app` (UID 1654) by default, no shell, smaller attack surface than
`mcr.microsoft.com/dotnet/aspnet:9.0`.

```dockerfile
# syntax=docker/dockerfile:1.7
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:9.0@sha256:<digest> AS build
WORKDIR /src
COPY ["Directory.Packages.props", "global.json", "App.sln", "./"]
COPY ["src/App.Api/App.Api.csproj", "src/App.Api/"]
COPY ["src/App.Application/App.Application.csproj", "src/App.Application/"]
COPY ["src/App.Domain/App.Domain.csproj", "src/App.Domain/"]
COPY ["src/App.Infrastructure/App.Infrastructure.csproj", "src/App.Infrastructure/"]
RUN dotnet restore "src/App.Api/App.Api.csproj"
COPY src/ src/
RUN dotnet publish "src/App.Api/App.Api.csproj" \
    -c Release -o /app/publish \
    --no-restore --runtime linux-x64 --self-contained false \
    -p:PublishReadyToRun=true

# Runtime stage — chiseled, runs as non-root by default
FROM mcr.microsoft.com/dotnet/aspnet:9.0-jammy-chiseled@sha256:<digest> AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENTRYPOINT ["dotnet", "App.Api.dll"]
```

Replace `<digest>` placeholders with values from
`docker pull mcr.microsoft.com/dotnet/sdk:9.0` then `docker images --digests`.
Pin in production manifests.

`.dockerignore`:

```
**/bin
**/obj
**/.vs
**/.idea
**/*.user
.git
.github
tests
*.md
```

## Makefile

```makefile
.PHONY: restore build test lint run docker-build docker-up migrate seed clean

restore:
	dotnet restore

build:
	dotnet build --no-restore -c Release

test:
	dotnet test --no-build -c Release --collect:"XPlat Code Coverage"

lint:
	dotnet format --verify-no-changes

run:
	dotnet run --project src/App.Api

docker-build:
	docker build -t $(APP_NAME) .

docker-up:
	docker compose up -d

migrate:
	dotnet ef database update --project src/App.Infrastructure --startup-project src/App.Api

seed:
	dotnet run --project src/App.Api -- --seed

clean:
	dotnet clean
	rm -rf src/*/bin src/*/obj tests/*/bin tests/*/obj
```

## Common Pitfalls

- **`async void`** anywhere except event handlers. Always `async Task`.
- **`.Result` / `.Wait()`** on tasks → deadlocks under sync context. `await` it.
- **`DateTime.Now`** → use `DateTimeOffset.UtcNow` or inject `TimeProvider`
  (`.NET 8+`). Never use local time in persisted data.
- **`IDisposable` not disposed** — use `using` or `await using` for everything
  that implements it; especially `DbContext`, `HttpClient` factories'
  scope, and `CancellationTokenSource`.
- **`HttpClient` per request** → socket exhaustion. Use
  `IHttpClientFactory` (`services.AddHttpClient<IUsersApi, UsersApiClient>()`).
- **EF Core N+1**: turn on `LogTo` in dev with `LogLevel.Information` for
  `Microsoft.EntityFrameworkCore.Database.Command`. Watch for repeated
  identical queries.
