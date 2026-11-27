# 01 — Recommended modernization strategy

## Executive recommendation

Use a **contract-first, test-first, hybrid modernization strategy implemented through complete vertical slices**.

Do not default to a full greenfield rewrite. Do not default to a large in-place refactor either. First inspect the actual API and decide which boundary should remain stable and which internals should change.

The default target should be:

1. Preserve the existing public API contract and deployment boundary.
2. Keep the existing API host, middleware, authentication, configuration and observability where they are already correct.
3. Create a clean application/use-case structure behind the existing API boundary.
4. Introduce only the abstractions needed to isolate external dependencies and enable tests.
5. Implement one complete representative business capability end to end.
6. Validate its functional parity against the existing implementation.
7. Repeat for the remaining capabilities in dependency-aware order.
8. Remove obsolete packages and infrastructure only after no production code depends on them.

A separate new API project or deployable host is an option, not the default. The target may be a new logical application architecture inside the current API solution/project. A second host is justified only when the assessment shows a strong reason and the organization can safely operate, test and deploy both implementations during migration.

## Why this approach is the best balance

| Concern | Full in-place refactor | Full greenfield rewrite | Recommended hybrid / vertical slices |
|---|---|---|---|
| Preserves existing host and integration setup | Usually | Often requires recreation | Usually |
| Allows a cleaner internal design | Limited by existing structure | Yes | Yes, incrementally |
| Exposes hidden business behaviour early | Depends on tests | Risk of discovering it late | Yes, slice by slice |
| Supports incremental parity validation | Yes | Possible, but often postponed | Core design feature |
| Deployment and rollback complexity | Lower | Potentially high | Controlled; no second host unless justified |
| Risk of accumulating fake implementations | Low | High if skeleton is overbuilt | Low when fakes are test-only |
| Ability to stop after a validated slice | Yes | Less natural | Yes |
| Cost of duplicated operational infrastructure | Low | Potentially high | Kept low by default |

## What “fresh skeleton” should mean

The useful part of the fresh-skeleton idea is a **clean target architecture and contract surface**, not necessarily a new deployable application.

The skeleton can be built in the existing solution, behind existing endpoints, or as a separate project if the decision gate approves it. It should establish:

- Existing route/method/version and API schema compatibility.
- Request/response DTOs and error contract.
- Authentication, authorization and policy enforcement.
- Endpoint-to-use-case wiring.
- Dependency injection and configuration conventions.
- Exception handling, logging, telemetry and correlation.
- Cancellation propagation.
- Test seams for external dependencies.
- A contract test harness.

**Fakes belong in test composition.** They are not substitutes for unfinished production logic. Do not register a fake repository, dummy service or canned successful response in a deployment-capable production configuration. An unfinished endpoint must fail visibly in development/test or remain unrouted until its complete vertical slice is ready; it must not appear production-complete.

A skeleton can prove route registration, serialization, status codes, authentication setup and API documentation. It cannot prove that business logic, persistence, messaging, integrations or failure handling are correct.

## The three strategy choices

### A. Incremental in-place modernization

Choose this when:

- The current API host and project boundaries are sound.
- Controllers/endpoints can be separated from application logic.
- Existing tests can be expanded around current behaviours.
- The main problems are replaceable libraries, excessive coupling in selected areas or inefficient implementations.
- A new host would add deployment/operational cost without meaningful value.

Method: capture behaviour, strengthen tests, then refactor one component or use case at a time.

### B. Hybrid modernization — default recommendation

Choose this when:

- The existing external API and deployment boundary are useful and must remain stable.
- Internals are too coupled or inconsistent to modernize safely with small mechanical edits.
- New application/use-case structure would simplify the system.
- Existing and new implementation paths can coexist temporarily behind stable endpoints or explicit routing.

Method: keep the API boundary, create a clean target application structure, migrate one vertical slice at a time, and retire old handlers/services when each capability passes parity.

This is usually the best balance for an established production API.

### C. New API host/project

Choose this only when supported by evidence, for example:

- Existing hosting/startup configuration is fundamentally unsafe or unmaintainable.
- Legacy dependency constraints prevent the target architecture.
- Internal coupling makes incremental extraction more expensive or risky than a new host.
- There is a defined coexistence, routing, data and deployment plan.
- Contract parity and operational readiness can be tested independently.
- The team can support both systems during transition and has a clear rollback/retirement plan.

A new host does not justify changing the API contract or rewriting all business logic in one large batch. It still uses the same test-first, vertical-slice approach.

## Recommended logical architecture

Prefer a **modular application with feature-oriented vertical slices**, not layers created for their own sake.

An illustrative structure:

```text
src/
  Api.Host/
    Endpoints/ or Controllers/
    Middleware/
    Authentication/
    OpenApi/
    Composition/
  Application/
    Features/
      Customers/
        GetCustomer/
          GetCustomerRequest.cs
          GetCustomerResponse.cs
          GetCustomerUseCase.cs
          GetCustomerValidator.cs       # only if genuinely needed
      Assignments/
      Contracts/
    Abstractions/                       # only stable outbound ports
  Domain/                               # if domain rules merit a separate boundary
    Customers/
    Assignments/
  Infrastructure/
    Persistence/
    Messaging/
    ExternalServices/
    Resilience/
tests/
  Api.UnitTests/
  Api.IntegrationTests/
  Api.ContractTests/
  Api.AcceptanceTests/
  Api.ParityTests/
  Api.PerformanceTests/                 # only where justified
```

This is a logical example, not a mandatory project-per-folder prescription. For a modest API, keep the number of projects small and use feature folders. A separate Domain project is valuable when it creates a genuine dependency boundary, not simply because Clean Architecture diagrams show one.

Dependency direction should generally be:

- API host depends on Application and Infrastructure composition.
- Application depends on Domain and its own abstractions.
- Infrastructure implements Application-owned outbound abstractions and depends on Domain as needed.
- Domain does not depend on API, persistence, messaging or framework infrastructure.

Do not create a generic repository, unit-of-work, CQRS or mediator abstraction automatically. Use direct application services/use cases and specific interfaces only where they isolate a real external dependency or make a business rule independently testable.

## Vertical-slice definition

A vertical slice is a complete business capability from the HTTP boundary to the real business outcome. For a read use case it may include:

1. Endpoint/route and contract.
2. Authentication and authorization.
3. Input validation.
4. Application use case.
5. Business/domain rules.
6. Real persistence query.
7. Any required downstream read.
8. Response mapping and error handling.
9. Telemetry and cancellation.
10. Unit, integration, contract, acceptance and parity tests.

For a write use case, add transactional behaviour, concurrency/optimistic locking, idempotency, outbox/event publication and failure recovery as applicable.

A slice is not complete when only the endpoint or mock-backed handler exists.

## Pilot-first rule

Before migrating all endpoints, select one representative capability that:

- Is important enough to exercise real application architecture.
- Has manageable scope and test data.
- Exercises representative dependencies (for example database plus one external service, if suitable).
- Is not the most catastrophic/highest-risk operation for the first experiment.
- Has clear expected outcomes and a way to isolate side effects.

Implement the pilot completely and review whether the architecture, testing approach, abstractions, telemetry and parity harness work. Adjust the plan before scaling to remaining slices.

## What not to do

- Do not implement every endpoint as a dummy success response and postpone real behaviour until the end.
- Do not create a second host solely to make the code look new.
- Do not mirror the old class structure if it has no architectural value.
- Do not change API behaviour and package architecture in the same unreviewable batch.
- Do not put business rules in controllers/endpoints to avoid writing application services.
- Do not build a custom MediatR replacement or generic pipeline framework.
- Do not introduce abstractions for every class or every data access call.
- Do not call a green build “parity”.

## Success definition

The new implementation is successful when the required capabilities and externally relied-upon contracts are preserved, all meaningful side effects are correct, security and reliability remain sound, the full validation suite passes, and any performance/maintainability improvements are evidenced. Source-code similarity to the old API is not a success criterion.
