# 05 — Test strategy and end-to-end parity

## Principle

The existing API is the reference implementation for required and approved behaviour. The purpose of tests is to capture required behaviour and detect meaningful regressions—not to preserve every accidental bug without review.

A new API skeleton returning the same HTTP shape with dummy data has demonstrated **contract compatibility only**. It has not demonstrated functional, data, integration or operational parity.

## Test layers

Use the existing test framework and repository conventions unless a documented gap justifies a change.

| Test type | What it proves | Typical focus |
|---|---|---|
| Unit | Small business rules behave correctly | Domain rules, mapping, validator, edge cases |
| Component/application | A use case coordinates its collaborators correctly | Use-case flow, transaction intent, error mapping |
| API integration | Host and application work together | Routing, DI, middleware, serialization, auth |
| Contract | Public interface remains compatible | OpenAPI, schemas, status, headers, error contract |
| Infrastructure integration | Real persistence/messaging/client integration works | SQL/Cosmos, Service Bus, Redis, HTTP |
| Acceptance | A business scenario produces the expected result | End-to-end business workflow |
| Parity | New and reference paths produce equivalent outcomes | Same scenarios, outputs and side effects |
| Resilience/fault | Failure behaviour is correct | Timeout, retry, circuit breaker, dependency outage |
| Security | Access decisions remain correct | Anonymous, unauthorized, forbidden, scopes/claims |
| Concurrency/idempotency | Duplicate/competing operations are safe | Race, retry, optimistic concurrency, duplicate messages |
| Performance | Changes meet agreed operational budgets | Latency, throughput, CPU, allocations, GC |

Not every endpoint requires a separate test project for every category. Organize test suites so each category can be run and reported independently where useful.

## Baseline before change

Record:

- Commit/hash tested and SDK/runtime used.
- Restore/build results.
- Test suites and exact commands executed.
- Total/passed/failed/skipped counts and duration.
- Failure details and whether each is pre-existing.
- Relevant coverage data and exclusions.
- Configuration and infrastructure required.
- Representative request/response fixtures.
- Performance workload and environment details.

A test result without command, commit, environment and report artifact is weak evidence. Do not claim a test category passed if it was not executed.

## Contract parity matrix

For each endpoint/operation, compare at least:

- HTTP method, route and version.
- Request schema, required fields and validation.
- Response schema, field presence and nullability.
- Status code, headers and content type.
- JSON names, enum values, date/time, precision and casing.
- Empty/missing resource behaviour.
- Pagination/filter/sort/default semantics.
- Authentication and authorization decision.
- Error payload and public error code.

Use OpenAPI comparisons where available, but supplement with runtime tests. Schema equality alone cannot capture every behavioural rule.

## Functional parity matrix

For each important use case, compare:

- Business decision and resulting state.
- Data read/write and values changed.
- Transaction commit/rollback behaviour.
- Concurrency/version checks.
- Idempotency and duplicate-request outcomes.
- Events/messages published or consumed.
- Message payload/schema, routing and relevant metadata.
- Downstream HTTP requests, authentication, response handling and timeout behaviour.
- Cache writes/invalidations where externally relevant.
- Retry/circuit-breaker/fallback outcomes.
- Audit records and other material side effects.
- Cancellation and partial-failure behaviour.

A parity result must specify which outputs are compared and which are intentionally excluded.

## Parity harness design

Prefer one shared scenario definition that can run against both implementations. Depending on architecture, the harness may use:

- Two `HttpClient` instances pointed at isolated old/new test hosts.
- A shared test suite parameterized by the implementation under test.
- Recorded baseline fixtures for deterministic read-only behaviour.
- Test database snapshots or resettable databases.
- Test containers or approved emulators for infrastructure fidelity.
- Stub HTTP services that capture outbound requests.
- Test message brokers or recording publishers.
- State inspection after each operation.

The right option depends on the API and available environment. Do not force a new framework if current tooling can do this reliably.

### Safe side-effect rules

- Use isolated databases/schemas and unique test tenants/keys.
- Never run duplicate production writes to compare implementations.
- Never publish duplicate real messages from shadow traffic.
- Never call non-idempotent external operations twice to build a comparison.
- For outbound writes, use a controlled fake/recording service or a safe test environment.
- For read-only shadow comparisons, confirm reads have no hidden side effects (billing, access logs with business impact, rate-limit consumption, etc.).
- Avoid replaying real PII or secrets into an unapproved test environment.
- Ensure test cleanup is deterministic and cannot delete non-test data.

## Deterministic comparison

Normalize only fields that are genuinely nondeterministic and do not represent business behaviour, for example:

- Generated correlation/request identifiers.
- Timestamps only when generated at runtime and not part of the business rule under test.
- Randomized provider IDs only when the ID's value is not contractually meaningful.

Never normalize away:

- Monetary values/rounding.
- Date/time zone conversions.
- Ordering when ordering is part of the contract.
- Null versus missing properties.
- Validation error codes.
- Retry counts or message ordering when those are business/operational guarantees.
- Authorization results.
- Database state or event payload differences.

For timestamps, use fixed clocks or controlled inputs where possible rather than broad tolerances.

## Edge-case catalogue

Generate tests from the discovered API and business rules. Common categories include:

### Request and validation

- Missing body, malformed JSON and unsupported content type.
- Missing/null/empty fields.
- Boundary numeric values and string lengths.
- Invalid enum/date/format/culture.
- Duplicate collection entries.
- Nested objects, large collections and oversized payloads.
- Conditional field dependencies.
- Asynchronous/database-dependent validation.

### Business behaviour

- State transition allowed and disallowed.
- Missing entity, soft-deleted entity and inaccessible entity.
- Duplicate create/update.
- Partial update versus replace semantics.
- Concurrency/version conflict.
- Boundary dates, time zones and daylight-saving transitions where relevant.
- Monetary/decimal precision and rounding.
- Pagination first/last/empty/invalid continuation token.

### Security

- Anonymous, wrong identity, missing role/scope/claim.
- Cross-tenant/resource access.
- Token expiry and invalid audience/issuer where testable.
- Privilege escalation and indirect object reference.

### Dependency and reliability

- Timeout, connection refused, 429 and 5xx.
- Malformed/unexpected downstream payload.
- Retry exhaustion and circuit open/half-open.
- Database unavailable, transaction rollback and deadlock/concurrency exception.
- Message duplicate, poison message, dead-letter and redelivery.
- Cancellation before and during an operation.
- Partial failure after one side effect but before another.

### Operational

- Configuration absent/invalid.
- Startup failure and readiness/liveness.
- Telemetry and correlation propagation.
- Large payloads, high concurrency and resource pressure.

## Test gap prioritization

Score each gap using:

- Business criticality (1–5).
- Exposure/probability (1–5).
- Failure impact (1–5).
- Change surface / likelihood of regression (1–5).
- Detectability (inverse score if failures are hard to observe).

A numerical score is a discussion aid, not a mathematical proof. Prioritize money/data/security/message correctness before low-impact formatting edge cases.

## Integration-test guidance

For ASP.NET Core, `WebApplicationFactory<TEntryPoint>` and `TestServer` can provide in-process host integration tests. Use a test-specific host configuration and replace external adapters deliberately. This is not a substitute for tests against real infrastructure when provider behaviour matters.

Use the existing database emulator, local infrastructure or approved Testcontainers pattern where available. Ensure the test data and container lifecycle are isolated and repeatable.

## Test result policy

- A new failure blocks the affected plan.
- A pre-existing failure must be recorded, not silently treated as passing.
- A flaky test is not a pass; stabilize or quarantine only under an explicit policy with owner and expiry.
- Do not skip tests or weaken assertions to satisfy the pipeline.
- Do not use coverage percentage as the only measure of test quality.
- Preserve the intent of each existing assertion when replacing assertion libraries.
- Test names should describe scenario and expected outcome.

## Slice-level definition of done

A vertical slice is complete only when:

- Public contract remains approved and compatible.
- Actual production adapters are used for the capability.
- Business logic and persistence are implemented.
- Auth, validation, cancellation, errors and transactions are covered as relevant.
- Unit/component tests pass.
- Integration/contract/acceptance tests pass.
- Parity is passed or every difference has explicit approval.
- External side effects are safely verified.
- Logs/traces/metrics remain fit for operations.
- Plan, traceability and risk records are updated.
