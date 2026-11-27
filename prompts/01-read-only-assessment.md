# Prompt 01 — Read-only architecture, responsibility, contract and test assessment

> Run after Prompt 00. This prompt must not change application or test source code.

```text
Act as a Principal .NET Solution Architect, Senior Software Engineer, Test Architect, Integration Architect and Performance Engineer.

Context:
- The selected existing production Web API is already on .NET 10 (`net10.0`).
- Framework migration is explicitly out of scope. Do not upgrade to .NET 11.
- The goal is to preserve every required business responsibility and externally relied-upon API contract while simplifying architecture, reducing unnecessary dependencies, improving testability and measuring performance.
- We may choose in-place, hybrid, or a separate new API host, but no choice has been approved yet.

IMPORTANT: This is a READ-ONLY assessment. Do not edit application source, tests, project files, package references, configuration, pipelines or deployment assets. Documentation deliverables are allowed. Do not remove or upgrade packages.

PHASE A — Validate scope and repository safety
1. Confirm the current branch and working tree.
2. Confirm `net10.0`, selected .NET SDK and CI alignment.
3. Record the baseline commit.
4. Stop if the repository has changed unexpectedly or if the selected API/project scope is ambiguous.

PHASE B — Discover all responsibilities
Inspect the complete relevant solution, not only controllers. Include:
- Controllers, Minimal API endpoints, route groups, API versioning and OpenAPI.
- Request/response DTOs, mapping, serialization and error contracts.
- Application services, MediatR requests/handlers/notifications/behaviours.
- Domain entities, business rules, state transitions and invariants.
- Validators, filters, middleware, exception handlers and decorators.
- Dependency injection, assembly scanning, service lifetimes and open generics.
- Persistence/data access, transactions, concurrency, indexes and migrations.
- Cache operations, keys, expiration and invalidation.
- Message publishers/consumers, queues/topics, schemas, delivery, retry and idempotency.
- External HTTP clients and other upstream/downstream dependencies.
- Background services, scheduled jobs, timers, file storage and event processing.
- Authentication, authorization, claims, scopes, tenant/resource access rules.
- Configuration, feature flags, secrets references, environment overrides.
- Logging, metrics, tracing, correlation, health/readiness and diagnostics.
- Build scripts, test configuration, CI/CD and deployment/runtime assumptions.

Trace each API operation to its actual business outcome. Also find non-HTTP triggers and responsibilities that could be lost in a rewrite.

Create:
1. API endpoint/operation catalogue.
2. Application responsibility matrix.
3. Business rules and validation catalogue.
4. Upstream/downstream dependency map.
5. Data-flow and side-effect map.
6. Security/authorization matrix.
7. Existing package and usage inventory, including direct/transitive package status where discoverable.
8. Unknowns, assumptions and questions requiring human owners.

For each endpoint or trigger, record: stable ID, purpose, code path, inputs, outputs, validation, auth, data reads/writes, transactions, events/messages, downstream calls, error/failure behaviour, tests and criticality.

PHASE C — API contract and observable behaviour
Inspect OpenAPI and implementation. Record method, route, version, request schema, response schema, status codes, headers, content types, error envelope, null/missing behaviour, serialization/casing, date/time, numeric precision, pagination/filter/sort defaults and authorization. Identify discrepancies between documentation and runtime behaviour.

Classify observed behaviour as:
- Required/contractual behaviour to preserve.
- Intentional current design.
- Accidental behaviour or probable defect requiring a decision.
- Unknown requiring confirmation.

Do not assume all historical behaviour is desirable. Do not silently change it either.

PHASE D — Existing test baseline
Identify all test projects and test categories. Determine the actual test commands/pipeline stages; do not assume `dotnet test` runs everything.
Execute the existing safe test/build commands if the environment allows. Record exact command, commit, SDK, environment, passed/failed/skipped counts, duration, warnings and artifact paths. Distinguish pre-existing failures from new failures. Do not modify tests to make them pass.

Assess coverage gaps across:
- Unit and domain rules.
- Application/use case behaviour.
- API integration and serialization.
- Contract and backward compatibility.
- Persistence/provider behaviour and transactions.
- Message publishing/consumption.
- External dependency integration.
- Acceptance and end-to-end workflows.
- Auth/security and tenant isolation.
- Concurrency/idempotency.
- Resilience, timeouts and partial failures.
- Performance and memory.

For each meaningful gap, identify production risk, scenario, suitable test layer, priority and required test fixture.

PHASE E — Performance baseline
Do not assume the API is slow. Identify representative critical endpoints and existing telemetry. Where feasible, run or design a controlled baseline. Record workload, payload, concurrency, environment and metrics. Capture latency (p50/p95/p99), throughput, CPU, memory/allocations, GC, DB query count/duration and dependency latency where available. Mark unmeasured hypotheses as hypotheses.

PHASE F — Existing architecture constraints
Assess coupling, cyclic dependencies, project structure, handler complexity, DI patterns, test seams, shared packages, deployment boundary, data ownership and operational constraints. Identify which parts should be preserved and which are reasonable modernization targets.

DELIVERABLES
Create under `docs/modernization/`:
- 01-architecture-assessment.md
- 02-application-responsibility-matrix.md
- 03-api-contract-inventory.md
- 04-upstream-downstream-dependencies.md
- 05-data-flow-and-side-effects.md
- 06-existing-test-baseline.md
- 07-test-gap-analysis.md
- 08-performance-baseline.md
- risks-and-unknowns.md

Use concrete file paths, class/method names and configuration keys. Do not include secrets or production personal data. Do not fabricate missing details. If a deliverable is not applicable, say why.

FINAL RESPONSE
Summarize application scope, responsibilities, API contract, dependencies, tests, known failures, critical gaps, performance evidence and unknowns. Do not propose a final architecture yet; that is the next phase. Confirm that no source/test/package/configuration files were changed.
```
