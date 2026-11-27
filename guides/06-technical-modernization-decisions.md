# 06 — Technical modernization decisions

## Starting position

The APIs are already on .NET 10. Treat that as a fixed constraint for this initiative. Do not add a framework upgrade to the scope or upgrade to .NET 11 merely because it is newer. Use the team's approved .NET 10 servicing patch and SDK consistently across developer machines and CI. Check current support/patch information when work begins; pin the intended SDK where appropriate.

.NET 10 supports C# 14. Use features available in the project's actual SDK/compiler and avoid preview features unless explicitly approved. New language syntax is not automatically a performance or reliability improvement.

## 1. MediatR replacement

### Recommendation

Prefer explicit application services/use-case classes invoked directly through constructor-injected dependencies. Do not replace MediatR with another mediator package or build an in-house mediator framework unless a separately approved requirement demonstrates a real need for message dispatch semantics.

### Inventory before removal

Inspect every usage of:

- `IRequest<T>`, `IRequest`, `IRequestHandler<,>` and `IRequestHandler<>`.
- `INotification`, notification handlers and publishers.
- Pipeline behaviours.
- Pre-processors and post-processors.
- Exception handlers.
- Custom wrappers, decorators and assembly scanning.
- Open-generic registrations and DI lifetime rules.
- Cancellation token propagation.

For every behaviour, identify its actual responsibility and move it to the right boundary:

- Validation → API boundary or application use case, depending on rule type.
- Transaction management → use-case/persistence transaction boundary.
- Authorization → endpoint policy and/or application/domain enforcement as appropriate.
- Logging/tracing → host/instrumentation or explicit business event logging.
- Caching → explicit query/service boundary with preserved invalidation.
- Idempotency → application/persistence boundary.
- Notifications → explicit application workflow, domain event handling or durable messaging as required.
- Exception mapping → application error model and central HTTP mapping.

### Target pattern

A use case should expose one clear method such as `ExecuteAsync` or a domain-meaningful method. Inject only what it needs. The endpoint calls it directly. Use request/response records/classes where they express the operation's data contract, not because MediatR requires them.

### Important caveats

- Do not lose hidden behaviour in pipeline behaviours.
- Do not change transaction boundaries or notification timing.
- Do not convert in-process notifications into distributed messages (or vice versa) without an explicit design decision.
- Do not change handler lifetimes without checking scoped dependencies.
- Do not make controller/endpoint classes responsible for complex business orchestration.
- Avoid one giant application service. Keep use cases focused.
- Avoid a `RequestDispatcher`, `ICommandBus`, generic `Send` or custom reflection dispatcher that recreates the removed abstraction.

### Completion condition

No MediatR production references, package registrations, scanning or hidden package behaviour remain. Test references and transitive dependency status are understood. All use cases pass behaviour and parity tests.

## 2. FluentValidation replacement or retention

### Recommendation

Do not remove FluentValidation solely to reduce package count. Inventory rule semantics and decide rule by rule. Replace it when native or explicit alternatives are simpler and genuinely equivalent. Retain it when complex validation is clearer and safer than a hand-built replacement; document that as a deliberate choice.

### Classify rules by responsibility

| Rule type | Preferred place to evaluate |
|---|---|
| Required field, format, range, simple request shape | API input validation / DataAnnotations or native framework validation |
| Nested object and collection shape | Native validation only after verifying nested/collection semantics |
| Cross-field request rule | Request validator or application boundary |
| Async check against database/external system | Explicit asynchronous application validator or use-case check |
| Permission/access rule | Authorization policy plus resource-level check |
| Business invariant/state transition | Domain model/service; must not rely only on HTTP validation |
| Uniqueness/availability condition | Application/domain with database constraint as final integrity protection |

.NET 10 has `Microsoft.Extensions.Validation` support for Minimal API validation using DataAnnotations and `IValidatableObject`, including asynchronous validation interfaces. That guidance is not a blanket drop-in replacement for every MVC or FluentValidation scenario. Verify the actual app style and framework support.

### Preserve exactly where contractually relevant

- Validation trigger point and order.
- Cascade/stop behaviour.
- Conditional rules.
- Nested and collection validation.
- Async validation.
- Rule sets and selectors.
- Localization.
- Error codes, messages, property paths and response envelope.
- Dependency access and cancellation.

Do not move a database-dependent validator into an attribute that cannot safely receive the required dependency. Do not move domain rules into API DTO attributes.

### Recommendation for working with MediatR

Keep the work packages separately testable. First extract and test validation behaviour at the appropriate boundary while preserving current FluentValidation rules if needed. Then remove MediatR dispatch/behaviours. Finally decide whether FluentValidation itself should be removed. If the repository's dependency graph makes a different order safer, document it.

## 3. FluentAssertions replacement

### Recommendation

Prefer the project's existing test framework assertions:

- xUnit: `Assert` APIs.
- NUnit: `Assert` / constraint model.
- MSTest: `Assert` and `CollectionAssert` APIs.

Use Shouldly only when the existing framework genuinely cannot provide the needed readability or diagnostic quality and the team approves the dependency. Do not introduce a second assertion ecosystem just for style.

### Migration rules

- Translate assertions by intent, not only by syntax.
- Preserve exception type and message checks where meaningful.
- Preserve deep/object equivalency semantics and collection ordering semantics.
- Check async exception assertions and cancellation behaviour.
- Retain useful failure messages.
- Do not weaken comparisons, reduce expected values or delete difficult tests.
- Run the affected suite before/after and compare test inventory.

A test package migration should not include unrelated production-code refactoring.

## 4. Polly and Microsoft.Extensions resilience

### Recommendation

Replace direct Polly usage in application code with Microsoft's `Microsoft.Extensions.Resilience` and/or `Microsoft.Extensions.Http.Resilience` integration when that improves configuration, hosting/DI integration and observability. Preserve custom strategies only when justified.

**Important:** These Microsoft packages are built on Polly. This modernization changes how the application integrates/configures resilience; it does not necessarily remove Polly from the transitive dependency graph. Do not claim “Polly has been eliminated” solely because direct code references are gone.

### Inventory current semantics

For each outbound dependency, record:

- Which operations are retried and which are not.
- Retry count, delay, exponential backoff, jitter and max delay.
- Which status codes/exceptions trigger a retry.
- Timeout per attempt and total operation budget.
- Circuit-breaker failure ratio, sampling duration, minimum throughput and break duration.
- Fallback behaviour and its correctness.
- Rate limiting/concurrency isolation/hedging if used.
- Strategy order and nesting.
- Cancellation and telemetry.
- Idempotency and deduplication safeguards.

### Safety rules

- Do not retry non-idempotent operations unless idempotency keys, deduplication or equivalent safeguards make it safe.
- Avoid retry at multiple layers (API client, service, gateway, caller) multiplying attempts.
- Preserve the end-to-end timeout budget; inner timeouts must not silently exceed caller deadlines.
- Distinguish transient from permanent failures.
- Do not use fallback data that turns dependency failure into an incorrect success.
- Keep resilience configuration per dependency where thresholds differ.
- Test retry/circuit-breaker behaviour with controlled failure injection.

## 5. .NET 10 / C# 14 modernization

Use the capabilities available in the actual .NET 10 SDK/C# 14 compiler, but adopt changes selectively. Keep `TargetFramework` at `net10.0` throughout this initiative.

Candidates to evaluate:

- C# 14 syntax/features where they improve clarity or remove boilerplate.
- `System.Text.Json` source generation for measured serialization hot paths or trimming/AOT constraints.
- Efficient async patterns and cancellation propagation.
- `IAsyncEnumerable<T>` for streaming when consumers benefit and resource lifetimes are safe.
- `Span<T>` / `ReadOnlySpan<T>` for synchronous parsing and buffer operations.
- `Memory<T>` / `ReadOnlyMemory<T>` when memory must be represented across async boundaries.
- Avoiding unnecessary LINQ materialization or repeated enumeration in hot paths.
- Efficient collection sizing and collection choices.
- `ArrayPool<T>` only where allocation pressure is measured and ownership/disposal are rigorously handled.
- Source generators or compiled accessors only when they solve a measured issue or maintainability concern.
- Runtime/JIT/library improvements supplied by the .NET 10 runtime without unnecessary source churn.

### Performance cautions

- Records are not inherently faster or more memory-efficient than classes. Choose records for value-based data semantics and concise immutable models; choose classes when reference identity/mutability/lifetime semantics fit better.
- `record struct` and other structs can increase copying, boxing or generic complexity when large or used incorrectly.
- `Span<T>` is stack-only/ref-like and cannot be retained across arbitrary async suspension; use `Memory<T>` where ownership/lifetime crosses async calls.
- Pooling adds ownership, clearing, concurrency and disposal complexity. Use it only after profiling.
- Avoid premature `ValueTask<T>` adoption; it can complicate APIs and should be justified by a hot path with a meaningful synchronous-completion rate.
- Do not hand-optimize loops when database/network latency dominates.
- Do not use unsafe code, pinning or custom memory management without a strong measured case and review.

### Performance method

For representative endpoints, compare the same workload and environment. Capture:

- Throughput (requests/second).
- p50, p95 and p99 latency.
- CPU consumption.
- Working set / managed heap where available.
- Allocated bytes per request/operation.
- Gen 0/1/2 GC counts and pause information where available.
- Exception/error rate.
- Database query count and duration.
- External dependency latency.

Use `dotnet-counters`, `dotnet-trace`, existing APM/OpenTelemetry, load-testing tools and microbenchmarks as appropriate. BenchmarkDotNet is useful for isolated code benchmarks, not a replacement for API/load testing. Ensure warmup, representative payloads, concurrency, data volume, runtime, environment and configuration are comparable.

## 6. Testing technology decisions

- Keep the current test runner unless there is a demonstrated limitation.
- Keep tests separated by purpose sufficiently to run fast unit tests independently from infrastructure tests.
- Use `WebApplicationFactory<TEntryPoint>` / `TestServer` for HTTP integration scenarios where the host model allows.
- Use real databases, emulators, containers or provider-specific integration tests for semantics that in-memory substitutes cannot prove.
- Keep fakes deterministic and explicit. A mock of a repository does not prove SQL query translation, transaction behaviour or provider semantics.
- Do not test EF Core query correctness exclusively with an in-memory fake provider.
- Do not add Playwright to a pure API test suite unless a browser/UI client is part of the scope.

## 7. Package and framework version control

- Inventory direct and transitive package references before changes.
- Use a consistent version-management approach already adopted by the repo (for example Central Package Management if already present).
- Review package security advisories and compatibility before upgrades.
- Avoid broad package upgrades mixed into a behaviour migration.
- Check whether a package remains transitively required.
- Do not manually remove a transitive dependency by forcing an incompatible version.
- Confirm local, CI and deployment runtime compatibility.
- Keep `net10.0`; framework version is not an incidental modernization variable here.

## Official references

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- [C# 14 features](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14)
- [.NET 10 overview](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview)
- [ASP.NET Core validation in .NET 10](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation?view=aspnetcore-10.0)
- [Resilience in .NET](https://learn.microsoft.com/en-us/dotnet/core/resilience/)
- [Diagnostics overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/)
- [`dotnet-counters`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-counters)
- [`dotnet-trace`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)
