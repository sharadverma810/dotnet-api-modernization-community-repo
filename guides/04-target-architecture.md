# 04 — Target architecture and design guidance

## Design objective

Design for explicitness, testability and low accidental complexity. The target should be easier for a human or AI agent to understand than the current implementation. Do not optimize for maximum layers, interfaces, design patterns or project count.

## Recommended structure

Prefer feature-oriented organization for use cases and thin technical boundaries for shared infrastructure. A modest API may use fewer projects and folders than this example.

```text
src/
  Api.Host/
    Controllers/ or Endpoints/
    Middleware/
    Authorization/
    OpenApi/
    DependencyInjection/
  Application/
    Features/
      FeatureA/
        UseCaseA/
          Request.cs
          Response.cs
          UseCase.cs
          Validator.cs                 # only when necessary
      FeatureB/
    Abstractions/
  Domain/                               # only if it contains meaningful domain rules
  Infrastructure/
    Persistence/
    Messaging/
    ExternalClients/
    Caching/
    Resilience/
tests/
  UnitTests/
  IntegrationTests/
  ContractTests/
  AcceptanceTests/
  ParityTests/
  PerformanceTests/                    # optional
```

### Dependency rules

- Endpoints/controllers translate HTTP concerns into application calls; they should not own business workflows.
- Application use cases coordinate a business operation and express required dependencies.
- Domain code owns stable business invariants and domain behaviour where the domain is sufficiently rich.
- Infrastructure implements outbound capabilities such as database access, message publishing and external clients.
- Keep framework and transport types out of the Domain layer.
- Do not expose persistence entities as public API DTOs unless the repository has a deliberate, reviewed contract policy that permits it.
- Avoid cyclic project references.
- Avoid a generic repository over EF Core or another data technology unless it solves a demonstrated problem.

## Direct application services versus mediator

Use direct, explicit use-case invocation as the default:

```csharp
app.MapGet("/customers/{id:int}", async (
    int id,
    GetCustomerUseCase useCase,
    CancellationToken cancellationToken) =>
{
    var result = await useCase.ExecuteAsync(id, cancellationToken);
    return result is null ? Results.NotFound() : Results.Ok(result);
});
```

The use-case class should be explicit and focused. Constructor injection is enough for ordinary dependency management. Do not build a `IMediator`, generic message bus, reflection dispatcher, auto-registration framework or custom pipeline simply to avoid writing a method call.

A request/response class is still useful when it models a real use case. CQRS can be used as a modelling choice without requiring MediatR or separate read/write databases.

## Cross-cutting behaviour placement

Map each existing behaviour to the right boundary:

| Behaviour | Likely location | Important caveat |
|---|---|---|
| HTTP input validation | API boundary / validation component | Preserve existing response shape and order if contractual |
| Application preconditions | Use case | Do not confuse with domain invariants |
| Domain invariants | Domain model/service | Must protect all entry points, not only HTTP |
| Authorization | Endpoint/policy and application/domain checks where required | Never rely solely on UI/route visibility |
| Transactions | Application use-case boundary / persistence unit of work | Preserve atomicity and isolation |
| Idempotency | Application/infrastructure boundary | Required for retried commands and message consumers where applicable |
| Logging/tracing | Host and explicit instrumentation | Avoid logging secrets/PII or duplicate noisy logs |
| Error mapping | Central HTTP exception mapping plus meaningful application errors | Do not leak internals |
| Caching | Explicit query/service boundary | Preserve invalidation and consistency semantics |
| Resilience | Outbound client/operation boundary | Avoid retry at multiple stacked layers |
| Message publication | Application workflow + outbox/transactional pattern where required | Preserve delivery and ordering guarantees |

Do not preserve a MediatR pipeline merely by copying it into a new generic pipeline. Identify the responsibility, decide its natural boundary, and test it.

## API contract preservation

Capture and preserve:
- Route templates, HTTP verbs and API versions.
- Request/response schema, required/optional fields and null semantics.
- Enum representation, casing, date/time formats, numeric precision and serialization settings.
- Status codes, headers, content types and error envelope.
- Validation error codes/messages/order when clients rely on them.
- Pagination, filtering, sorting and default values.
- Authorization schemes, policies and claims/scopes.
- Idempotency and concurrency semantics.

Do not assume semantically similar JSON is compatible. Check clients and generated SDK assumptions where possible.

## Data, transactions and messaging

- Do not change database schema as incidental cleanup. Database changes require their own migration plan and compatibility assessment.
- Keep old and new code compatible with the same schema during any period of coexistence.
- For expand/contract migrations: add backward-compatible schema first, deploy compatible readers/writers, migrate data, then remove old schema only after consumers are proven migrated.
- Preserve transaction boundaries, isolation levels, optimistic concurrency and unique constraints.
- For messages, preserve schema, routing, ordering, partition/session behavior, duplicate handling, dead-letter behaviour and retry semantics.
- Where a database update and message publication must be atomic, assess an outbox or the existing equivalent. Do not casually change delivery guarantees.
- Never send duplicate real messages from a parity harness.

## Dependency injection and composition

- Register concrete application services explicitly or with the repository's established pattern.
- Prefer explicit registration when it improves discoverability and avoids accidental assembly scanning.
- Validate scopes and lifetimes; do not inject scoped dependencies into singleton services.
- Avoid service locator patterns and resolving dependencies manually inside business logic.
- Keep production and test composition visibly separate.
- Add a test that ensures fake/test registrations cannot appear in production composition where this risk exists.

## Error handling and cancellation

- Preserve cancellation semantics and pass `CancellationToken` through meaningful async boundaries.
- Do not convert cancellation into a generic 500 response.
- Distinguish validation, not-found, conflict, unauthorized/forbidden, dependency failure and unexpected internal errors.
- Preserve public error shape while avoiding internal stack traces and secret values.
- Do not swallow exceptions or return success-shaped fallback data unless that is an explicit existing contract and approved behaviour.

## Observability

Retain or improve:
- Structured logs with stable event names and relevant identifiers.
- Distributed trace propagation across HTTP, database and messaging boundaries.
- Metrics for request rate, duration, errors and dependency health.
- Correlation/trace IDs with clear propagation rules.
- Health checks that distinguish process liveness from dependency readiness.

Avoid logging tokens, credentials, sensitive payloads or unnecessarily identifiable data. Compare observable behaviour where operations teams depend on it.

## API style

Do not migrate Controllers to Minimal APIs (or the reverse) as a side effect of removing MediatR. Keep the existing style unless a specific plan proves a benefit. API style is not the objective; correct contracts and maintainable implementation are.
