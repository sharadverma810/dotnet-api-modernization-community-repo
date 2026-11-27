# .NET 10 Web API Modernization Playbook — Complete Markdown Edition

> This file consolidates the complete reusable playbook. The companion ZIP contains the same content split into individual documents for easier use.

---

<!-- BEGIN DOCUMENT: README.md -->

## .NET 10 Web API Modernization Playbook

**Reusable, contract-first, test-first modernization framework for existing production .NET Web APIs**  
**Version:** 1.0  
**Prepared:** 27 November 2026  
**Target framework:** Existing .NET 10 applications (no framework migration in scope)

## What this playbook is for

Use this playbook to modernize one existing .NET Web API at a time while preserving its required responsibilities, API contracts, data behaviour, security, integrations and operational characteristics. It is designed for AI-assisted engineering with GitHub Copilot Agent or another repository-aware coding agent.

The recommended method is **contract-first, test-first, hybrid modernization using complete vertical slices**. It deliberately avoids both extremes: blindly refactoring a legacy implementation in place and automatically creating a second deployable API that reimplements everything from scratch.

## Download options

- `DOTNET-API-MODERNIZATION-PLAYBOOK.md` — all guides, prompts and templates combined into one Markdown file.
- `dotnet-api-modernization-playbook.zip` — the complete folder structure with each document separated for easier reuse.

## Start here

1. Read [01 — Recommended strategy](../guides/01-recommended-strategy.md).
2. Read [02 — Stages and decision gates](../guides/02-stages-and-gates.md).
3. Follow [03 — Implementation order](../guides/03-implementation-order.md).
4. Apply the non-negotiable rules in [07 — Engineering rules and AI guardrails](../guides/07-engineering-rules-and-ai-guardrails.md).
5. Use the prompts in numerical order, beginning with [Prompt 00 — Project preflight](../prompts/00-project-preflight.md).

## Recommended workflow

| Step | Prompt | Expected result | Code changes? |
|---|---|---|---:|
| 0 | `00-project-preflight.md` | Safe project scope, branch and repository state | Branch only, after checks |
| 1 | `01-read-only-assessment.md` | Architecture, responsibility, contract, dependency and test baseline | No |
| 2 | `02-strategy-and-plan.md` | Evidence-based strategy decision and ordered plans | Documentation only |
| 3 | `03-contract-first-skeleton.md` | Contract test harness and target skeleton, only if justified | Limited, gated |
| 4 | `04-pilot-and-vertical-slices.md` | One pilot capability, then complete slices | Yes, one slice at a time |
| 5 | `05-dependency-modernization.md` | Safe package / pattern migrations | Yes, one concern at a time |
| 6 | `06-final-validation-and-dev-readiness.md` | Full validation, parity evidence and DEV readiness pack | Documentation and test fixes |
| 7 | `07-independent-review.md` | Adversarial review of evidence, gaps and go/no-go recommendation | Review first; fixes only if approved |

The prompts are reusable. They are not a command to run everything unsupervised. Respect each approval gate and the actual repository's test/deployment capabilities.

## Key recommendation in one paragraph

Keep the existing API as the reference implementation. First understand and document the behaviour that must be preserved. Build a reliable contract and parity test harness. Choose the least risky target architecture: usually keep the existing API host, routes and deployment boundary while simplifying internals; create a separate API host only when evidence justifies the cost and migration risk. Establish a clean target structure, prove it with one representative end-to-end vertical slice using real infrastructure adapters and controlled test dependencies, then migrate remaining capabilities one by one. Modernize MediatR, FluentValidation, FluentAssertions and Polly in separate, evidence-based plans so cross-cutting behaviours are not lost. Measure performance before claiming an improvement. Finish with complete regression, integration, parity, security, operational and deployment-readiness validation.

## Scope assumptions

- All in-scope Web API projects have already been migrated to `net10.0` with minimal changes.
- .NET framework migration is **out of scope**. Do not change the target framework to .NET 11 or another version as part of this initiative.
- Use the approved .NET 10 SDK/runtime servicing level used by local development and CI. Pin the SDK consistently where the repository uses `global.json`.
- Apply this process to one API/project or one bounded modernization unit per feature branch. Coordinate shared libraries separately.
- The repository, existing tests, CI pipeline and current production behaviour are authoritative evidence. Do not invent missing requirements.

## Folder guide

- `guides/` — strategy, stages, implementation order, architecture, parity, technical decisions, guardrails and deliverables.
- `prompts/` — copy/paste instructions for an AI coding agent.
- `templates/` — repeatable planning, parity, risk and progress templates.

## Important distinction

**Contract parity is not functional parity.** An endpoint that returns the same JSON using dummy data has only demonstrated a portion of HTTP contract compatibility. Functional parity also requires the intended business decision, data changes, transaction semantics, messages/events, downstream calls, authorization and failure behaviour to match.

## Official technical references

The technical guidance in this playbook is informed by these Microsoft references (verify versions again when executing the work):

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- [What's new in C# 14](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14)
- [What's new in .NET 10](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview)
- [ASP.NET Core validation in .NET 10](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/validation?view=aspnetcore-10.0)
- [Resilience in .NET](https://learn.microsoft.com/en-us/dotnet/core/resilience/)
- [ASP.NET Core integration testing](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0)
- [.NET diagnostics overview](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/)
- [`dotnet-counters`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-counters)
- [`dotnet-trace`](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)

---

**Operating principle:** Modernize the implementation, not the externally relied-upon responsibilities. Every change must be traceable to an approved plan, protected by tests, and supported by execution evidence.

<!-- END DOCUMENT: README.md -->

---

<!-- BEGIN DOCUMENT: guides/01-recommended-strategy.md -->

## 01 — Recommended modernization strategy

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

<!-- END DOCUMENT: guides/01-recommended-strategy.md -->

---

<!-- BEGIN DOCUMENT: guides/02-stages-and-gates.md -->

## 02 — Modernization stages and decision gates

This lifecycle is intended to be repeatable for each API. The AI agent must not skip a gate because the next task appears straightforward.

## Stage 0 — Scope, safety and baseline branch

**Goal:** Establish a safe, isolated work area and define what is in scope.

Activities:

- Identify repository, solution, API project(s), shared libraries and runtime entry point.
- Confirm the project already targets `net10.0`; do not change target framework or turn this work into a .NET 11 upgrade.
- Check SDK, `global.json`, CI SDK configuration and deployment runtime compatibility.
- Inspect Git branch, working tree, untracked files and current commit.
- Confirm the latest intended `main` reference is available locally/remotely according to team practice.
- Create a unique feature branch from the intended `main` only after verifying the working tree is safe.
- Never overwrite an existing branch or discard local changes.
- Define the project boundary and any shared-library impacts.

Deliverables: scope note, branch name, baseline commit, repository status and known blockers.

**Gate 0:** No source changes until branch safety and scope are confirmed. If the tree is dirty, preserve changes and stop branch operations that could affect them.

## Stage 1 — Read-only architecture and responsibility discovery

**Goal:** Understand the API as it operates today, not as documentation says it should operate.

Inspect:

- Routes, controllers/Minimal APIs, API versions, DTOs and OpenAPI.
- Request/response mappings, status codes, headers, errors and serialization options.
- Authentication, authorization, claims, scopes and policies.
- Business rules, state transitions, validation and permission checks.
- Application services, MediatR handlers/behaviours, events and domain logic.
- Persistence, transactions, concurrency, indexes, data ownership and migrations.
- Cache semantics, keys, TTL and invalidation.
- Messaging publishers/consumers, schemas, delivery guarantees and idempotency.
- External HTTP clients, authentication, timeouts and resilience strategies.
- Background jobs, scheduled work, timers and file operations.
- Middleware, filters, logging, tracing, metrics and health checks.
- Configurations, secrets references, environment differences and feature flags.
- Test projects, CI/CD, deployment manifests and runtime dependencies.

Create an endpoint responsibility matrix, upstream/downstream map, data-flow map and unknowns register. Follow each endpoint into the code that produces its real outcome. Identify hidden responsibilities embedded in handlers, filters, validators, middleware, repositories and package behaviours.

Deliverables: architecture assessment, responsibility catalogue, API contract inventory, dependency map, data-flow notes, package usage inventory and known-unknowns list.

**Gate 1:** Every in-scope endpoint and major background/message responsibility has an owner/path through the code, or an explicit unknown is recorded. No implementation changes yet.

## Stage 2 — Existing behaviour, test and performance baseline

**Goal:** Create reproducible evidence of current behaviour and system health.

Activities:

- Run restore/build and all currently configured tests on the baseline.
- Record pass/fail/skipped counts, durations, warnings and known pre-existing failures.
- Identify which test categories actually run in the pipeline; do not assume one `dotnet test` command covers everything.
- Capture representative API response examples and OpenAPI contracts.
- Characterize edge cases, error payloads, auth rules, ordering, pagination, time zones and null/empty semantics.
- Capture meaningful data state changes, published events/messages and downstream effects using controlled test fixtures.
- Record performance baseline under a repeatable workload for important endpoints.
- Record environment/configuration needed to reproduce tests.

Add characterization tests for existing intended behaviour where gaps would make later refactoring unsafe. Do not blindly encode known defects: classify each as required compatibility, accidental behaviour, defect or approved future change.

Deliverables: test baseline, coverage gap analysis, contract fixtures, behaviour matrix, parity strategy, performance baseline and pre-existing-failure register.

**Gate 2:** There is enough test and behavioural evidence to detect regressions in the next planned slice. Critical gaps are either closed or explicitly accepted as residual risk by the responsible reviewer.

## Stage 3 — Select strategy and approve target architecture

**Goal:** Decide in-place, hybrid or new host/project based on repository evidence.

Activities:

- Compare the three options using complexity, contract risk, coupling, host quality, testability, operational cost and rollback.
- Default to hybrid: preserve outward API/deployment boundary, improve internals.
- Define desired dependency direction and feature/use-case structure.
- Define where validation, authorization, transactions, idempotency, caching, logging and resilience live.
- Map old components to target components and identify temporary coexistence.
- Define data compatibility and deployment sequencing.
- Define which package removals are independent and which rely on extracted behaviours.

Deliverables: architecture decision record (ADR), target architecture, migration map, risk register, ordered plans, rollback strategy and acceptance criteria.

**Gate 3:** Human/technical owner approves the strategy, plan order, scope and compatibility assumptions. AI must not self-approve material architecture or business changes.

## Stage 4 — Contract-first skeleton and parity harness

**Goal:** Establish the target boundary and prove tests can exercise both old and new paths.

Activities:

- Build the smallest target skeleton justified by Stage 3.
- Preserve route/method/version, request/response DTO shape, error model and auth policy.
- Configure test host and fake outbound adapters only in the test composition.
- Create contract tests that run against existing and target implementations.
- Create a parity runner or shared scenario suite.
- Provide test fixtures for database, message broker and external systems.
- Ensure production registration cannot accidentally select fakes.

If existing host is retained, the skeleton can be a new application feature structure within the same API project/solution. It does not require a second deployable service.

Deliverables: target skeleton, contract tests, test composition, fake adapters, parity harness and configuration safeguards.

**Gate 4:** Contract test infrastructure works against the baseline and target. Fakes are isolated to test-only configuration. Skeleton is not called functionally complete and is not deployed as a production replacement.

## Stage 5 — Pilot vertical slice

**Goal:** Validate the proposed architecture with one complete, representative use case.

Activities:

- Select pilot from the dependency-aware capability map.
- Implement actual application logic, persistence and required integrations.
- Keep DTO/API boundary stable.
- Preserve error, cancellation, transaction and security behaviour.
- Add unit, integration, contract, acceptance and parity tests.
- Compare side effects and performance where relevant.
- Review whether abstractions and folder/project structure helped or added needless complexity.

Deliverables: production-capable pilot slice, test evidence, parity report, performance comparison and architecture feedback/ADR updates.

**Gate 5:** Pilot satisfies all slice acceptance criteria. Reviewers approve scaling the pattern or require adjustment before more slices are started.

## Stage 6 — Sequential vertical-slice implementation

**Goal:** Migrate remaining capabilities in the approved dependency order.

For each slice:

1. Read its plan and responsibility/contract evidence.
2. Confirm prerequisites and test fixtures.
3. Implement only the slice and essential shared infrastructure.
4. Run focused unit/component tests.
5. Run relevant integration, contract, acceptance and parity scenarios.
6. Compare data state and external effects.
7. Review security, concurrency, cancellation and failure paths.
8. Measure performance if the slice changes a performance-sensitive path.
9. Update traceability, documentation and plan status.
10. Stop on unexplained regression; do not advance to dependent slices.

Old and new paths may coexist during migration. Avoid duplicate processing or writes. Switch a capability only through a controlled routing/feature-flag mechanism where one exists and is safe.

Deliverables: completed slices, traceability matrix, updated parity matrix and retired-old-code list.

**Gate 6:** Every migrated slice has passed its acceptance and parity criteria. No dependent plan starts on a failed slice.

## Stage 7 — Cross-cutting package and architecture cleanup

**Goal:** Remove direct dependencies and obsolete patterns after replacement behaviour is proven.

Potential workstreams:

- MediatR extraction/removal.
- FluentValidation replacement/retention decision.
- FluentAssertions replacement.
- Direct Polly API replacement with Microsoft.Extensions resilience integration where appropriate.
- Dead code, package and DI registration cleanup.
- Consistent exception/validation/observability policies.

Do not bundle all package removals into one commit or plan. Some can be executed during slice migration, but complete package removal only after usage scans show no remaining references.

Deliverables: package inventory diff, code-reference scan, dependency graph review and validation evidence.

**Gate 7:** No unintended direct references remain; transitive dependencies are understood; behaviour and pipeline ordering remain correct.

## Stage 8 — Full verification and operational readiness

**Goal:** Prove the complete API is a credible DEV deployment candidate.

Run all applicable tests and checks: build, unit, component, integration, contract, acceptance, end-to-end, parity, authorization/security, concurrency/idempotency, resilience/fault, migration/data compatibility and performance.

Review:

- DEV configuration and secret references.
- Database/schema migration order and backward compatibility.
- Messaging compatibility and consumer/producer rollout order.
- Health/readiness checks.
- Logs, metrics, traces and alerts.
- Deployment slots/rolling strategy and rollback.
- Feature flags and capability switching.
- CI/CD and artifact consistency.

Deliverables: final verification report, test evidence, parity report, performance report, deployment checklist, rollback plan and QA handover.

**Gate 8:** Human release owner decides whether DEV deployment is approved. A green test report alone is not deployment authorization.

## Stage 9 — DEV deployment and QA feedback

**Goal:** Validate the candidate in the target DEV environment before QA proceeds.

Activities:

- Deploy only with explicit authorization.
- Verify startup, health, configuration, auth, connectivity and telemetry.
- Run DEV smoke tests and safe integration scenarios.
- Reconcile logs, database and message outcomes.
- Record QA findings and defects.
- Fix and retest using the same traceability and parity rules.

Do not call a DEV deployment proof of production safety. DEV is an additional validation environment, not a substitute for UAT/STG/production release controls.

**Gate 9:** QA and technical owners accept the DEV results or document remaining issues and next action.

<!-- END DOCUMENT: guides/02-stages-and-gates.md -->

---

<!-- BEGIN DOCUMENT: guides/03-implementation-order.md -->

## 03 — Recommended implementation order

This is the default sequence. The assessment may adjust the order based on actual dependencies, but it must explain every deviation. The sequence separates enabling work from code modernization so that the team does not remove packages before understanding the behaviours they implement.

## Priority definitions

- **P0 — Safety prerequisite:** Required before architectural changes or deployment decisions.
- **P1 — Foundational:** Establishes the target architecture or removes a significant dependency safely.
- **P2 — Capability migration:** Moves business responsibilities to the target implementation.
- **P3 — Optimization/cleanup:** Evidence-based efficiency and maintainability improvements after functional correctness is established.

## Master order

| Order | Priority | Work package | Main output | Must precede |
|---:|---|---|---|---|
| 0 | P0 | Scope, branch and .NET 10 baseline | Safe feature branch; no framework migration | All work |
| 1 | P0 | Architecture and responsibility discovery | Responsibility/dependency/contract inventory | All design and code changes |
| 2 | P0 | Existing test and behaviour baseline | Pass/fail baseline, characterization cases | Refactoring |
| 3 | P0 | Test gap closure for critical behaviour | Risk-based test additions | Affected refactors |
| 4 | P0 | Contract/parity harness | Shared scenarios and comparator | Skeleton and vertical slices |
| 5 | P0 | Architecture option decision | Approved in-place/hybrid/new-host ADR | Skeleton |
| 6 | P1 | Minimal target skeleton and composition | Testable contract boundary; safe fake adapters | Pilot |
| 7 | P1 | Pilot vertical slice | One complete real business capability | Scale-out |
| 8 | P1/P2 | Remaining vertical slices | Complete migrated capabilities | Final package cleanup |
| 9 | P1 | FluentAssertions replacement (if required) | Existing test framework assertions; optional Shouldly | Not a business prerequisite; do after baseline |
| 10 | P1/P2 | MediatR behaviour extraction and removal | Explicit use-case invocation; preserved pipeline semantics | Remove package after all usages gone |
| 11 | P2 | FluentValidation decision and migration | Native/explicit validation or documented retention | May be staged with MediatR extraction, not blindly bundled |
| 12 | P2 | Direct Polly usage modernization | Microsoft.Extensions resilience integration where useful | Retry/timeout parity tests |
| 13 | P2 | Cross-cutting cleanup | Remove dead registrations, handlers, packages, code | Full validation |
| 14 | P3 | Measured performance and memory improvements | Benchmarks/profile evidence and code improvements | Performance baseline |
| 15 | P0 | Full regression, parity and security verification | Complete validation report | DEV readiness |
| 16 | P0 | DEV readiness and QA handover | Deployment/rollback/config checklist | Authorized DEV deployment |

The numbering is a planning aid, not an instruction to perform every package migration globally before implementing any slice. For a large API, remove MediatR use case by use case as slices move to the new design, while old handlers remain for unmigrated capabilities. Retire the package only when all references and behaviours have been accounted for.

## Detailed order and rationale

### 0. Establish scope and branch safety

- Confirm the exact API project and all shared projects it uses.
- Confirm the target is already `net10.0`; do not include .NET 11 upgrade work.
- Establish baseline commit and clean/dirty worktree state.
- Create one unique feature branch from the intended `main`.
- No reset, clean, stash, discard or force operation without explicit instruction.

### 1. Discover responsibilities before choosing a design

Inventory endpoints, background triggers, event consumers, outbound integrations, database writes, validation, security, side effects and cross-cutting behaviours. Trace all MediatR behaviours and custom pipeline code. Capture upstream/downstream contracts.

### 2. Record the baseline

Run the full existing test matrix and record pre-existing failures. Capture representative success, invalid-input, not-found, unauthorized, forbidden, conflict, dependency-failure and boundary scenarios. Confirm that the current test harness can reliably exercise real boundaries.

### 3. Close critical test gaps

Prioritize missing tests based on business impact and change risk, not arbitrary coverage percentage. Add tests around transactions, authorization, idempotency, retries, pagination, concurrency and failure recovery when relevant.

### 4. Build contract and parity harness

Prefer reusing existing test frameworks and tools. Create the same scenario suite usable against old and new implementations. Add deterministic fixtures and side-effect capture. Use `WebApplicationFactory` for ASP.NET Core integration tests where it fits the hosting model; use real or containerized infrastructure when the integration semantics require it.

### 5. Choose the target architecture

Use the decision matrix in Guide 01. Make a documented choice. Do not create a second host by default.

### 6. Build minimal skeleton

Implement only the architecture and test seams needed for the first complete slice. Do not generate dozens of placeholder endpoints or fake production responses. Verify API route/auth/serialization contract tests.

### 7. Prove the approach with a pilot

Choose a representative, bounded use case. Complete its end-to-end implementation. Validate business outcomes and side effects against the reference. Reassess design before scaling.

### 8. Implement remaining capabilities

Use dependency order, not alphabetic endpoint order. Usually migrate foundational reads and shared data access before dependent writes or workflows, but the actual order must be derived from the responsibility graph. For each slice, finish the complete definition of done before moving to a dependent capability.

### 9. Replace FluentAssertions

This is generally a test-only migration and can be isolated after the baseline. Prefer the assertion APIs already supplied by xUnit, NUnit or MSTest. Use Shouldly only if the existing framework cannot provide sufficient clarity and the team approves the dependency. Preserve assertion coverage and failure intent; do not merely translate syntax mechanically.

### 10. Remove MediatR safely

Map every request, handler, notification, behaviour, pre/post processor and exception handler. Extract validation, transactions, logging, caching, authorization, idempotency and other behaviours to their correct boundaries. Replace dispatch with explicit method calls/use-case classes. Avoid creating a homegrown mediator or generic pipeline to recreate the old package.

### 11. Decide on FluentValidation

Do a rules inventory before choosing a replacement. Move API input shape validation, application checks and domain invariants to appropriate layers. Use native validation only where it is semantically equivalent. Keep explicit validators for complex or asynchronous rules when appropriate. The package can be retained if replacement creates more risk or complexity than value; the objective is a better design, not package count.

### 12. Modernize direct Polly use

Use Microsoft.Extensions resilience packages for supported integration patterns when they simplify configuration and observability. Preserve current retry/timeout/circuit-breaker semantics, prevent retries of non-idempotent operations unless protected, and avoid nested retry strategies. Remember these packages are built on Polly; they remove direct integration from application code, not necessarily the transitive Polly dependency.

### 13. Remove obsolete code and dependencies

Search direct and transitive package usage, DI registrations, configuration, reflection registrations, assembly scanning and tests. Remove only after proving no capability depends on it.

### 14. Optimize performance only from evidence

Profile high-value endpoints and hot paths. Change code only where measurement or a clear algorithmic defect supports the change. Avoid Span/Memory/pooling/structs used merely as a style preference.

### 15–16. Full validation and DEV readiness

Run the complete matrix and prepare an evidence-backed deployment pack. Deployment is a separate approval action.

## How to reorder plans safely

A plan may move earlier only when:

- Its prerequisites are satisfied.
- Its scope is independent or dependencies are explicitly handled.
- Baseline and regression tests protect the affected behaviour.
- Its acceptance criteria remain testable.
- The new order does not create unsafe coexistence or data changes.

A plan may move later when:

- Its expected benefit is low relative to risk.
- A prerequisite has not been validated.
- Current code does not use the package/pattern.
- A replacement is not clearly better.

Every reordering must be recorded in the decision log with reason and impact.

<!-- END DOCUMENT: guides/03-implementation-order.md -->

---

<!-- BEGIN DOCUMENT: guides/04-target-architecture.md -->

## 04 — Target architecture and design guidance

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

<!-- END DOCUMENT: guides/04-target-architecture.md -->

---

<!-- BEGIN DOCUMENT: guides/05-parity-and-testing.md -->

## 05 — Test strategy and end-to-end parity

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

<!-- END DOCUMENT: guides/05-parity-and-testing.md -->

---

<!-- BEGIN DOCUMENT: guides/06-technical-modernization-decisions.md -->

## 06 — Technical modernization decisions

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

<!-- END DOCUMENT: guides/06-technical-modernization-decisions.md -->

---

<!-- BEGIN DOCUMENT: guides/07-engineering-rules-and-ai-guardrails.md -->

## 07 — Engineering rules and AI guardrails

These rules apply to every AI prompt and every implementation plan. Treat them as non-negotiable unless a named human owner approves a documented exception.

## A. Repository and Git safety

1. Work on one API modernization scope per feature branch by default. Shared-library changes must be included only when their impact is understood and approved.
2. Inspect `git status`, current branch and current commit before any operation.
3. Create the feature branch from the intended `main` reference only when the working tree is safe.
4. Never overwrite a branch with the same name. Choose a unique name or stop.
5. Never run destructive Git operations (`reset --hard`, `clean -fd`, force checkout, force push, history rewrite) unless the user explicitly requests the exact operation and understands the effect.
6. Never discard, overwrite or stash existing user work without explicit approval.
7. Do not push, merge, create a PR or deploy unless explicitly requested/authorized.
8. Keep changes scoped. Do not perform unrelated formatting or cleanup that obscures the modernization diff.
9. Record the base commit and branch in the assessment.

## B. Evidence and truthfulness

1. Inspect actual source code, configuration and tests. Do not infer behaviour solely from class names or README files.
2. Cite concrete file paths, types and methods in analysis documents.
3. Distinguish facts, assumptions, hypotheses and unknowns.
4. Never claim a test, build, benchmark, parity comparison or deployment succeeded unless it actually ran and produced evidence.
5. Record the exact command, commit, environment and results.
6. Preserve pre-existing test failures in the baseline register; do not silently attribute them to new code or claim they were fixed.
7. Do not invent missing requirements. Create an unknown/question with an owner.
8. Do not claim “production safe” or “zero risk”. State verified evidence and residual risks.

## C. Behaviour and compatibility

1. Existing intended business responsibilities and approved external contracts are the reference.
2. Distinguish required compatibility from accidental behaviour and defects.
3. Do not change route, HTTP verb, API version, DTO schema, status code, header, authorization, serialization, error envelope or pagination semantics without approval.
4. Do not normalize away a meaningful parity difference.
5. Do not remove hidden behaviour from middleware, filters, MediatR behaviours, validators, decorators, repositories or hosted services without replacing and testing it.
6. Preserve data ownership, transaction boundaries, concurrency, idempotency and message delivery semantics.
7. Preserve cancellation and timeout budgets.
8. Do not perform schema/data migrations as incidental refactoring.

## D. Test integrity

1. Characterize important existing behaviour before changing it.
2. Add or update tests in the same plan/slice that changes behaviour.
3. Never delete, skip, weaken or loosen an assertion merely to obtain a green build.
4. Do not mark a flaky test as passed; follow a documented quarantine policy with owner and expiry if unavoidable.
5. Run focused tests after each change and the relevant full regression suite before declaring a plan complete.
6. Contract tests alone do not prove functional parity.
7. Unit tests with mocked repositories do not prove real database semantics.
8. Use isolated test infrastructure and deterministic data.
9. No duplicate production writes, messages or non-idempotent calls for shadow/parity testing.
10. Record intentional differences and obtain approval before updating expected results.

## E. Architecture and code quality

1. Prefer explicit direct use-case/application-service calls over a new mediator abstraction.
2. Do not build a custom framework merely to replace a removed package.
3. Do not add a project, layer, interface, generic repository or abstraction without a demonstrated reason.
4. Keep business decisions out of controllers/endpoints and infrastructure adapters.
5. Keep domain invariants enforceable outside the HTTP path.
6. Use dependency inversion for real external dependencies, not every internal method call.
7. Avoid service locator, hidden global state and static mutable state.
8. Avoid synchronous blocking over asynchronous I/O.
9. Propagate cancellation to I/O operations.
10. Keep logs useful, structured and free of secrets/credentials/sensitive payloads.

## F. Dependency modernization

1. Inventory every package usage and transitive dependency before removal.
2. Replace one concern at a time.
3. Map and preserve MediatR pipeline behaviours.
4. Do not assume native validation is equivalent to FluentValidation.
5. Prefer native assertions from the existing test framework; add Shouldly only with justification.
6. Microsoft.Extensions resilience packages use Polly internally; distinguish direct integration removal from transitive package removal.
7. Review retry safety and idempotency before changing resilience.
8. Remove unused package references only after source, configuration, DI and runtime paths are checked.
9. Avoid unrelated package upgrades and target-framework changes.

## G. Performance and modern language features

1. Capture a baseline before optimization.
2. Optimize measured hot paths and algorithmic bottlenecks first.
3. Do not use Span, Memory, records, structs, pooling, ValueTask or unsafe code simply because they are available.
4. Assess lifetime, ownership, copying, boxing, thread safety and readability.
5. Benchmark the same workload under comparable conditions.
6. Report both improvements and regressions; never cherry-pick a single metric.
7. Performance changes must not weaken correctness, validation, observability or security.

## H. Security and data protection

1. Do not print, copy or commit secrets, tokens, certificates, personal data or production payloads.
2. Use environment variables, secret stores and approved test secrets.
3. Never connect test automation to production data without explicit approval and a safe data-handling plan.
4. Maintain authentication, authorization, tenant isolation and resource-level access controls.
5. Treat logs, snapshots and parity artifacts as potentially sensitive; redact and retain them according to policy.
6. Avoid test adapters that can be activated accidentally in production.
7. Check dependency security/advisories as part of final validation.

## I. Plan and approval discipline

1. Assessment is read-only.
2. Architecture and plan documents require review before skeleton/production-code work.
3. Each slice has objective, scope, non-goals, acceptance criteria, tests, risks and rollback.
4. A failing gate blocks dependent work.
5. Human approval is required for API/business contract changes, database/schema changes, auth changes, new deployment topology, data migrations, material resilience changes and DEV deployment.
6. The agent may proceed automatically only within an approved plan and its non-controversial implementation steps.
7. Update progress and traceability as work is completed.
8. Stop and report when a necessary decision cannot be inferred safely.

## J. Definition of done

A plan is done only when its acceptance criteria are evidenced, relevant tests pass, behavioural differences are understood and approved, documentation is updated, and the reviewer can reproduce the validation. “Code compiled” is not a definition of done.

<!-- END DOCUMENT: guides/07-engineering-rules-and-ai-guardrails.md -->

---

<!-- BEGIN DOCUMENT: guides/08-deliverables-and-project-workspace.md -->

## 08 — Deliverables and recommended workspace

## Recommended documentation workspace

Create this structure in the repository after the scope and branch are confirmed:

```text
 docs/
   modernization/
     README.md
     00-scope-and-baseline.md
     01-architecture-assessment.md
     02-application-responsibility-matrix.md
     03-api-contract-inventory.md
     04-upstream-downstream-dependencies.md
     05-data-flow-and-side-effects.md
     06-existing-test-baseline.md
     07-test-gap-analysis.md
     08-performance-baseline.md
     09-strategy-decision.md
     10-target-architecture.md
     11-modernization-plans.md
     12-implementation-order.md
     13-parity-testing-strategy.md
     14-production-readiness-criteria.md
     decision-log.md
     progress.md
     risks-and-unknowns.md
     plans/
       PLAN-001-<name>.md
       PLAN-002-<name>.md
     parity/
       endpoint-parity-matrix.md
       functional-parity-matrix.md
       test-evidence-index.md
     reports/
       baseline-test-report.md
       pilot-slice-report.md
       final-validation-report.md
       dev-readiness-report.md
```

The list is a recommended set. Do not create empty files merely to match a template. Create a document when relevant information exists and mark any not-applicable item with a reason.

## Core deliverables and contents

| Deliverable | Must contain |
|---|---|
| Scope and baseline | Repository, solution, API project, branch, base commit, SDK, target framework, environments, out-of-scope items |
| Architecture assessment | Current components, key code paths, layering, hotspots, known constraints |
| Responsibility matrix | Every endpoint/capability, business purpose, rules, data owner, dependencies, implementation path |
| API contract inventory | Routes, methods, versions, DTOs, status codes, errors, auth, serialization, pagination |
| Dependency map | Upstream/downstream system, protocol, operation, criticality, failure behaviour, owner |
| Data flow and side effects | Reads/writes, transaction, messages, cache, external calls, idempotency |
| Test baseline | Exact commands, environment, pass/fail/skip counts, existing failures, artifacts |
| Test gap analysis | Untested risk, impact, scenario, test layer and priority |
| Performance baseline | Workload, environment, p50/p95/p99, throughput, CPU, allocations, GC, query/dependency timing |
| Strategy decision | Option comparison and selected approach with rationale |
| Target architecture | Structure, dependency direction, behavior placement, transition path |
| Modernization plans | Independent plan descriptions with dependencies and acceptance criteria |
| Implementation order | Dependency-aware order, priority, gates and reason |
| Parity strategy | Scenario source, normalization, side-effect safety, comparator and acceptance |
| Readiness criteria | Test, security, configuration, data, deployment, monitoring and rollback gates |
| Decision log | Date, context, options, decision, rationale, consequences, approver |
| Progress | Plan state, tests executed, evidence links, blockers and next work |

## Plan status model

Use a small, explicit vocabulary:

- `NOT_STARTED`
- `ASSESSMENT_COMPLETE`
- `APPROVED`
- `IN_PROGRESS`
- `BLOCKED`
- `IMPLEMENTED_AWAITING_VALIDATION`
- `VALIDATION_FAILED`
- `PARITY_FAILED`
- `COMPLETE`
- `DEFERRED`
- `REJECTED`

Do not use `COMPLETE` when only source code has changed. A plan is complete only after validation and evidence.

## Traceability model

Every required responsibility should be traceable across:

`Responsibility ID → API/trigger → existing code path → target use case → dependency/side effect → test scenarios → parity evidence → plan ID → status`.

This prevents endpoints or non-HTTP work (for example hosted services and message consumers) from disappearing during a rewrite.

## Recommended AI execution rhythm

- Assessment: one broad read-only pass, then targeted follow-up for unknowns.
- Planning: propose options and plans; human review of material choices.
- Implementation: one plan/slice at a time; short feedback loop.
- Validation: run affected tests early, then the broader suite at gate boundaries.
- Review: run an independent adversarial review prompt after the final report.

## What to put in every plan

Use `templates/plan-template.md`. At minimum it must specify:

- Goal and business rationale.
- Existing responsibility and behaviour to preserve.
- Scope and non-goals.
- Dependencies and impacted files/components.
- Target design and implementation steps.
- API/data/security/resilience implications.
- Tests and parity scenarios.
- Acceptance criteria.
- Risks and rollback.
- Validation commands and evidence expected.

## Evidence storage

Keep test result files, coverage, logs, parity outputs and performance artifacts in the repository's approved artifact location. Do not commit large or sensitive outputs by default. Link to CI artifacts or sanitized reports. Never commit secrets, raw production data or sensitive customer payloads.

<!-- END DOCUMENT: guides/08-deliverables-and-project-workspace.md -->

---

<!-- BEGIN DOCUMENT: guides/09-multi-api-rollout.md -->

## 09 — Applying the playbook across multiple Web APIs

## Recommendation

Apply the method **one API modernization boundary at a time**, not as one portfolio-wide refactor across every API. You can standardize the discovery template, test approach, architecture principles and quality gates, but each API needs its own responsibility map, dependency graph, risk profile and parity suite.

## Portfolio-level inventory

Before selecting the first API, create a portfolio register:

| Field | Description |
|---|---|
| API ID/name | Stable internal name |
| Business capability | What business area it supports |
| Criticality | Business / customer / financial / operational impact |
| Traffic profile | Typical and peak request rates if known |
| Dependency count | Databases, queues, topics, caches, external services |
| Contract consumers | Internal/external APIs and client applications |
| Data ownership | Data stores and domain responsibility |
| Current architecture health | Coupling, complexity, testability, known risks |
| Test maturity | Coverage quality, integration and contract test availability |
| Operational health | Incidents, latency, error rates, observability |
| Shared-library impact | Common packages/components affected |
| Modernization status | Not assessed / assessed / approved / in progress / complete |

## Choosing the first API

Do not automatically choose the smallest API or the most complex API.

Choose a pilot API that is:

- Representative of the architecture patterns used by other APIs.
- Important enough to validate the playbook against real engineering constraints.
- Bounded enough that the team can complete assessment and one or two full slices.
- Supported by accessible test data and testable dependencies.
- Not so operationally risky that early architecture learning threatens a critical delivery.

A good pilot helps identify reusable standards (for example parity harness conventions, logging, validation approach and CI reports) without assuming every API has the same business architecture.

## Rollout sequence

1. Inventory all APIs and their shared dependencies.
2. Select one pilot API and apply the complete playbook.
3. Review the pilot's architecture, cost, defects, cycle time and test evidence.
4. Convert proven, genuinely reusable elements into shared templates/libraries only where appropriate.
5. Select the next API based on risk reduction and business priority.
6. Repeat per API; do not copy the pilot's folder structure or abstractions blindly.
7. Migrate shared libraries only with a separate compatibility plan and consumer inventory.

## Shared libraries

A shared library can affect many services. Before changing it:

- List all consumers and versions.
- Determine which APIs deploy together and which deploy independently.
- Preserve binary/source compatibility where required.
- Avoid changing all consumers in a single broad feature branch unless explicitly planned.
- Consider additive API evolution and versioning.
- Add consumer-level contract tests where practical.
- Coordinate package versions and release sequence.
- Keep shared library modernization separate from API capability migration when this improves isolation.

Do not introduce a new shared framework for application patterns solely to make all APIs look identical. Standardize cross-cutting infrastructure only when there is a real shared concern and clear ownership.

## Portfolio metrics

Track outcomes across APIs:

- Number of responsibilities mapped and migrated.
- Number of API/functional parity scenarios passed.
- New regression defects and escaped defects.
- Critical test gaps closed.
- Build/test pipeline duration and reliability.
- Package references removed or deliberately retained.
- Performance deltas on representative endpoints.
- Deployment/rollback issues.
- Maintenance complexity and team feedback.

Do not compare APIs solely by lines changed or package count. The strongest outcome is reduced operational and change risk with preserved business behaviour.

<!-- END DOCUMENT: guides/09-multi-api-rollout.md -->

---

<!-- BEGIN DOCUMENT: prompts/00-project-preflight.md -->

## Prompt 00 — Project preflight, scope and Git safety

> Run this first in the repository root. It establishes safe context; it is not permission to rewrite code.

```text
Act as a Principal .NET Solution Architect and repository modernization lead.

We are beginning a controlled modernization of an existing production .NET Web API. All in-scope APIs have already been migrated to .NET 10 with minimum changes. This initiative is NOT a framework upgrade. Keep TargetFramework at net10.0. Do not upgrade to .NET 11 or change the deployment runtime as part of this work.

Your task in this phase is project identification, scope definition and safe Git preflight only.

1. Identify the repository root, solution file(s), API project(s), test project(s), shared libraries and the likely runtime entry point(s).
2. Inspect each relevant .csproj, Directory.Build.props/targets, Directory.Packages.props, global.json, NuGet configuration and CI workflow enough to establish target framework and SDK alignment.
3. Confirm the selected API project targets net10.0. Report any mismatch; do not change it.
4. Inspect `git status --short --branch`, current branch, current commit and available main branch references. Do not assume the local `main` is current.
5. Check whether the working tree is clean. Include tracked, untracked and ignored files where relevant to avoid overwriting user work.
6. Propose a unique branch name: `feature/<api-short-name>-modernization-<work-item-or-date>`.
7. If and only if the working tree is safe, the intended base `main` reference is known, and the branch name does not already exist, create the feature branch from that base. Do not overwrite any branch.
8. If there are uncommitted changes, an uncertain base, a branch collision or any risk of losing work, do not switch branches or modify files. Report the exact blocker and safe next step.
9. Do not run `git reset`, `git clean`, force checkout, stash, force push, merge or rebase. Do not discard user changes.
10. Do not edit application source, tests, package references, configuration or deployment files. Do not push, create a PR or deploy.

Produce `docs/modernization/00-scope-and-baseline.md` only if the repository is safe to modify. Otherwise provide the same information in your response without creating or changing files.

Report:
- Repository and solution.
- API project(s) included and excluded.
- Shared project dependencies.
- Target framework and SDK evidence.
- Current branch, base commit and worktree status.
- Proposed/created feature branch.
- Risks or blockers.
- Exact commands executed and their outcomes.

Never claim that a branch was created unless the Git command succeeded and you verified the current branch afterward.
```

<!-- END DOCUMENT: prompts/00-project-preflight.md -->

---

<!-- BEGIN DOCUMENT: prompts/01-read-only-assessment.md -->

## Prompt 01 — Read-only architecture, responsibility, contract and test assessment

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

<!-- END DOCUMENT: prompts/01-read-only-assessment.md -->

---

<!-- BEGIN DOCUMENT: prompts/02-strategy-and-plan.md -->

## Prompt 02 — Architecture strategy decision and implementation planning

> Run after the read-only assessment has sufficient evidence. This phase may create or update planning documents but must not modify application source or tests.

```text
Act as a Principal .NET Solution Architect and modernization programme lead.

Read all existing documents under `docs/modernization/`, verify important claims against source code, and use the actual repository as the source of truth. The API already targets .NET 10. Do not add a framework upgrade to this initiative.

Your objective is to recommend the lowest-risk architecture and a dependency-aware set of independently testable implementation plans.

PART 1 — Validate the assessment
- Check that every exposed endpoint and non-HTTP trigger is represented.
- Check that MediatR handlers and pipeline behaviours are mapped.
- Check validators, transactions, caching, authorization, logging, notifications, idempotency and exception handling.
- Check direct/transitive package usage.
- Check that tests and baseline failures are correctly represented.
- Correct documentation errors only when source evidence is clear; record material corrections.
- Add unresolved items to risks-and-unknowns.md.

PART 2 — Compare three modernization strategies
A. Incremental modernization within existing API implementation.
B. Hybrid modernization: preserve existing host/public boundary, build a cleaner internal application structure and migrate by vertical slice.
C. New API host/project with contract-first skeleton and controlled coexistence.

Compare each for: API compatibility, coupling, ability to isolate dependencies, testability, business-risk discovery, performance, deployment complexity, observability, rollback, data/schema compatibility, team/operational cost and expected maintainability.

Do not choose C simply because a new skeleton looks cleaner. Do not choose A simply because it changes fewer files. Recommend the approach supported by repository evidence. The default starting hypothesis is B unless the assessment proves otherwise.

PART 3 — Define the target architecture
- Decide whether the host stays, whether new projects are warranted, and why.
- Define feature/use-case organization and dependency direction.
- Define endpoint-to-use-case interaction.
- Define validation, domain invariants, authorization, transactions, caching, messaging, resilience, logging and error mapping boundaries.
- Identify which abstractions isolate real infrastructure and which should not exist.
- Define temporary old/new coexistence and capability routing, if needed.
- Define data/schema and event/message compatibility constraints.
- Avoid a custom mediator/framework that recreates removed dependencies.

PART 4 — Decide what “skeleton” means
If appropriate, design a minimal contract-first target skeleton. It may exist inside the current host/solution; a second deployable host requires justification. Define only the routes, DTOs, auth, DI, configuration, observability, error handling and test seams needed to prove the boundary.

Fakes and dummy data must be test-only. Do not build all endpoints as successful dummy implementations and postpone real logic. A skeleton is not production-ready and cannot prove functional parity.

PART 5 — Create modernization plans
Create one plan per coherent workstream. At minimum evaluate:
- Test/parity harness and risk-based test gap closure.
- Minimal target architecture/skeleton if justified.
- Representative pilot vertical slice.
- Remaining business capability slices in dependency order.
- FluentAssertions replacement (test-only; use existing assertion framework first, Shouldly only if justified).
- MediatR behaviour extraction and direct use-case invocation.
- FluentValidation replacement or deliberate retention decision.
- Direct Polly usage migration to Microsoft.Extensions resilience integration where appropriate.
- Performance/memory optimization based on measured hot paths.
- Package/dead-code cleanup and final verification.

Keep package migrations distinct. Where validation is part of a MediatR behaviour, plan the safe extraction order; do not automatically combine MediatR and FluentValidation into one opaque change.

Each plan must contain:
1. ID/title/status/priority.
2. Business/technical rationale.
3. Existing behaviour and responsibilities to preserve.
4. Goal and measurable expected outcome.
5. Scope and non-goals.
6. Affected projects/files/components.
7. Prerequisites/dependencies.
8. Proposed design and implementation steps.
9. API/data/security/resilience implications.
10. Unit/component/integration/contract/acceptance/parity tests.
11. Performance validation when relevant.
12. Acceptance criteria and definition of done.
13. Risks, mitigations and rollback.
14. Validation commands and evidence expected.

PART 6 — Prioritize and sequence
Build a dependency-aware order. Prioritize prerequisites before refactors. Explain which plans may run in parallel and which must be sequential. A single API modernization should not become a multi-project big-bang change. State how shared libraries will be handled.

PART 7 — Approval gates
Identify decisions that require human approval, including API/business contract changes, schema/data migration, auth changes, new host/deployment topology, message contract changes, materially different retry/timeout semantics and DEV deployment.

DELIVERABLES
Create/update:
- 09-strategy-decision.md
- 10-target-architecture.md
- 11-modernization-plans.md
- 12-implementation-order.md
- 13-parity-testing-strategy.md
- 14-production-readiness-criteria.md
- `plans/PLAN-###-<name>.md` for each real plan
- decision-log.md
- risks-and-unknowns.md

FINAL RESPONSE
Provide the selected strategy and why the other options were not selected, target architecture, plan summary, sequence, risks, required approvals and measurable gates. Explicitly state whether a separate API host/project is recommended. Do not implement application code or modify tests in this phase.
```

<!-- END DOCUMENT: prompts/02-strategy-and-plan.md -->

---

<!-- BEGIN DOCUMENT: prompts/03-contract-first-skeleton.md -->

## Prompt 03 — Contract-first skeleton and parity harness

> Run only after Prompt 02 has been reviewed and the architecture/skeleton plan has been approved.

```text
Act as a Principal .NET Architect and senior implementation engineer.

Read the approved architecture decision, target architecture, plan, API contract inventory, responsibility matrix and parity strategy. The project is already on .NET 10; keep `TargetFramework` at `net10.0`.

Implement only the approved skeleton/parity-harness plan. Do not begin business capability migrations beyond the approved pilot setup. Do not create a new deployable API host unless the approved ADR explicitly selected it.

PRE-CHANGE SAFETY
1. Verify branch, baseline commit and working tree.
2. Confirm this plan is approved and prerequisites passed.
3. Run or verify the baseline build/test status.
4. List planned file changes and explain why each is needed.
5. Stop if there are unexpected local changes or a scope conflict.

SKELETON REQUIREMENTS
Preserve existing external API behaviour at the contract boundary:
- Routes, verbs and versions.
- Request/response DTO shape and serialization settings.
- Status codes, headers, content types and error contract.
- AuthN/AuthZ policy and claims expectations.
- Pagination/filter/sort/null/default semantics.
- Exception mapping, cancellation and correlation behaviour.

Use the approved target architecture. Prefer the current host and existing middleware/configuration if those are already correct. Create only the minimum project/folder boundaries required. Do not create generic abstractions or a project per layer without a reason.

TEST COMPOSITION
- Create or adapt contract tests that can run against the existing implementation and target skeleton.
- Use the existing test framework and test runner.
- Use `WebApplicationFactory<TEntryPoint>`/`TestServer` if appropriate for this ASP.NET Core host.
- Use deterministic test fixtures.
- Fake repositories, downstream clients and message publishers only in explicit test configuration.
- Ensure production DI/configuration cannot accidentally select fake/test adapters.
- Add a guard test or registration check if the repository's composition pattern makes fake activation a realistic risk.

IMPORTANT LIMITATION
A skeleton may prove route registration, serialization, DI composition, auth wiring, status/error mapping and contract compatibility. It does NOT prove business functionality, data correctness, transaction semantics, downstream integration or functional parity.

Do not create successful dummy implementations for every endpoint and call them production-complete. For an unfinished capability, use a test-only fake in its contract harness or leave its production path on the old implementation until a complete slice is ready. Do not expose placeholder successful responses in DEV/PROD.

PARITY HARNESS
- Use the same scenario definitions for old and new paths where technically possible.
- Capture request, response, status, headers and structured body.
- Define allowed normalization for truly nondeterministic fields only.
- Add controlled adapters for database state, outbound calls and message capture.
- Keep write scenarios in isolated test infrastructure.
- Never duplicate real production side effects.

VALIDATION
1. Build the affected projects.
2. Run the pre-existing baseline tests relevant to the change.
3. Run contract tests against both old implementation and target skeleton.
4. Run auth, error, serialization, cancellation and route tests.
5. Verify test-only fake registrations cannot leak into production.
6. Record exact commands and outcomes.
7. Review package changes; do not perform unrelated package upgrades.

DOCUMENTATION
Update skeleton plan status, target architecture, parity strategy, progress and decision log. Record limitations honestly.

STOP CONDITIONS
Stop if the contract inventory is incomplete, auth behaviour is unknown, fake wiring could reach production, contract differences are unexplained, or tests cannot isolate side effects safely.

FINAL RESPONSE
List changed files, implemented skeleton elements, contract-test coverage, exact test results, known gaps, fake adapter safeguards, deviations and whether the skeleton gate passed. Do not claim functional parity.
```

<!-- END DOCUMENT: prompts/03-contract-first-skeleton.md -->

---

<!-- BEGIN DOCUMENT: prompts/04-pilot-and-vertical-slices.md -->

## Prompt 04 — Pilot implementation and sequential vertical slices

> Run after the skeleton/parity harness gate passes. Use the same prompt for the pilot and each subsequent capability, updating the plan ID and slice scope.

```text
Act as the Principal .NET Solution Architect, Senior Application Engineer and Test Engineer responsible for migrating one approved business capability at a time.

Read:
- Approved target architecture and strategy ADR.
- Application responsibility matrix.
- API contract inventory.
- Upstream/downstream and data-flow maps.
- Test baseline and gap analysis.
- Parity strategy.
- The selected vertical-slice plan and dependency order.

The API is already on .NET 10. Keep `net10.0`. Do not widen the task to unrelated framework or package upgrades.

CURRENT SLICE
Plan ID: <PLAN-ID>
Capability / use case: <CAPABILITY>
In-scope operations/endpoints/triggers: <LIST OR REFERENCE TO PLAN>
Explicit non-goals: <LIST>

PRE-FLIGHT
1. Confirm current feature branch and clean/understood worktree.
2. Confirm prerequisites and previous slice gates passed.
3. Re-read the existing implementation path and the actual target path.
4. Confirm baseline test commands and known pre-existing failures.
5. Confirm test data and external side effects are isolated.
6. Identify any contract or business decision requiring human approval. Do not infer it.

IMPLEMENT A COMPLETE VERTICAL SLICE
The capability must be complete from its entry point to its real business outcome:
- Endpoint/route/trigger and DTO contract.
- Authentication, authorization and resource-level checks.
- Request validation at the correct boundary.
- Application use case and business rules.
- Domain invariants where applicable.
- Real persistence/data access.
- Real or controlled integration with required downstream dependencies.
- Transactions, concurrency and idempotency where required.
- Events/messages/cache side effects where required.
- Error mapping, logging, metrics, tracing and cancellation.
- Appropriate unit, component, integration, contract, acceptance and parity tests.

Do not stop at a mock-backed use case. Test fakes are appropriate for isolated unit/component scenarios; integration and acceptance scenarios must exercise the real provider/infrastructure behaviour where that behaviour matters.

IMPLEMENTATION RULES
- Preserve approved API contracts and intended business behaviour.
- Keep changes scoped to this capability and required shared infrastructure.
- Use direct application service/use-case calls unless the approved ADR states otherwise.
- Do not create a mediator framework or generic pipeline.
- Do not introduce unnecessary interfaces, generic repositories or layers.
- Do not make unrelated cleanup changes.
- Do not silently change validation response, error codes, transaction boundaries, retry rules or message contracts.
- Preserve CancellationToken propagation.
- Never add real credentials or production data to code/tests.

VALIDATION CYCLE
After each logical change:
1. Build the affected projects.
2. Run targeted unit/component tests.
3. Run integration tests against real/approved infrastructure or faithful provider tests.
4. Run contract tests.
5. Run acceptance scenarios.
6. Run parity scenarios against the reference implementation using identical controlled inputs.
7. Compare response/status/headers plus data state, transactions, outbound calls, messages, cache and audit effects as relevant.
8. Run security/authorization, concurrency/idempotency and failure tests relevant to this capability.
9. Run performance comparison if the slice changes a hot path or performance-sensitive behaviour.
10. Run the relevant regression suite.

PARITY SAFETY
- Do not invoke real non-idempotent downstream operations twice.
- Use isolated test data, fake/recording outbound adapters, controlled broker/database or safe test environment.
- Normalize only approved nondeterministic fields.
- Do not mask real mismatches with broad JSON ignore rules.
- Classify every difference as regression, intentional approved change, known legacy defect, nondeterminism or unresolved.
- Unexplained behaviour differences block completion.

SLICE ACCEPTANCE
Mark the slice complete only when:
- All in-scope responsibilities are implemented with real required behaviour.
- API contract is compatible or approved changes are recorded.
- Data, transaction, security and side-effect semantics are correct.
- Required tests pass.
- Parity passes or differences are explicitly approved.
- No unexplained regression exists.
- Logs/traces/metrics and cancellation are appropriate.
- Performance claims are supported by evidence.
- Plan, traceability, risk and progress documents are updated.

AFTER THE PILOT
If this is the pilot, explicitly assess whether the target architecture, test harness, abstractions, DI, observability and implementation cadence should be adjusted before scaling. Do not assume the pilot pattern should be copied blindly.

STOP CONDITIONS
Stop and report if tests cannot isolate side effects, a business rule is ambiguous, a schema/message/auth change is required, a parity gap cannot be explained, the target architecture proves inadequate, or a dependent plan prerequisite is incomplete.

FINAL RESPONSE
Report plan ID, slice, files changed, behaviour preserved, new behaviour implemented, tests and exact commands/results, parity comparison, performance evidence, risks, documentation updates and COMPLETE/BLOCKED/FAILED status. Do not begin the next slice unless the current slice's gate is passed.
```

<!-- END DOCUMENT: prompts/04-pilot-and-vertical-slices.md -->

---

<!-- BEGIN DOCUMENT: prompts/05-dependency-modernization.md -->

## Prompt 05 — Safe dependency and pattern modernization

> Run one technology workstream at a time. Replace `<WORKSTREAM>` with MediatR, FluentValidation, FluentAssertions or direct Polly integration. Do not combine multiple package removals into one opaque change.

```text
Act as a Principal .NET Architect and senior engineer specializing in safe dependency modernization.

WORKSTREAM: <WORKSTREAM>
PLAN ID: <PLAN-ID>

Read the approved plan, architecture decision, source usage inventory, responsibility matrix, test baseline, parity strategy and dependency order. The API is already on .NET 10. Keep `net10.0` and avoid unrelated package upgrades.

GENERAL RULES
1. Verify branch/worktree and plan approval.
2. Inspect actual direct and transitive package references and all usages.
3. Map the behaviour currently provided by the dependency before removing it.
4. Identify hidden behaviours, DI registration, reflection scanning, configuration, error handling, lifetimes and telemetry.
5. Implement the smallest safe change for this workstream only.
6. Preserve public API and business behaviour.
7. Run focused tests after each meaningful change, then relevant regression and parity tests.
8. Do not weaken assertions, tests or validation to achieve a green result.
9. Do not claim a package has been removed until source, configuration, DI, project and dependency graph checks confirm it.
10. Record any remaining transitive dependencies accurately.

WORKSTREAM-SPECIFIC GUIDANCE

IF WORKSTREAM = MEDIATR:
- Inventory all requests, commands, queries, handlers, notifications, pipeline behaviours, pre/post processors, exception handlers, decorators and registrations.
- Trace validation, transactions, authorization, logging, caching, idempotency, domain events and exception mapping currently executed through the pipeline.
- Extract each responsibility to its correct boundary and add tests before deleting dispatch code.
- Replace request dispatch with direct constructor-injected application use-case/service calls.
- Keep CQRS naming only where it clarifies the use case; do not require a mediator.
- Preserve notification timing/delivery and all transaction semantics.
- Do not build a custom mediator, generic `Send` dispatcher, reflection dispatch or generic pipeline to recreate MediatR.
- Remove the package only after all production references, behaviours and registrations are eliminated or deliberately retained elsewhere.

IF WORKSTREAM = FLUENTVALIDATION:
- Inventory every validator, rule, rule set, child/nested/collection rule, async rule, condition, cascade mode, custom validator, localization rule, error code and DI dependency.
- Classify each rule: request shape, cross-field application rule, async data check, authorization, or domain invariant.
- Consider native ASP.NET Core/.NET 10 validation, DataAnnotations, IValidatableObject, Microsoft.Extensions.Validation where supported, and small explicit asynchronous validators.
- Verify actual host style: Minimal API and MVC/Razor validation support are not interchangeable assumptions.
- Preserve timing, rule order, property path, error code/message and response envelope when contractual.
- Keep DB-dependent checks in an appropriate application boundary and retain database constraints for integrity.
- Do not move domain invariants only into API DTO annotations.
- If replacement is more complex or risky, document why FluentValidation is retained rather than removing it for package-count reasons.

IF WORKSTREAM = FLUENTASSERTIONS:
- Keep the existing test framework.
- Prefer its native assertion API (xUnit, NUnit or MSTest as applicable).
- Consider Shouldly only when the framework cannot provide sufficient clarity/diagnostics and approval exists.
- Migrate by test category and preserve assertion intent, equivalency/deep comparison, ordering, exception and async semantics.
- Do not delete or weaken tests. Do not change production code in this workstream.
- Compare test inventory and execution results before and after.

IF WORKSTREAM = POLLY:
- Inventory every policy/pipeline: retry predicates/count/delay/jitter, circuit breaker thresholds, timeout budgets, fallback, rate limit, hedging, strategy order and cancellation.
- Evaluate `Microsoft.Extensions.Resilience` and `Microsoft.Extensions.Http.Resilience` for DI/HttpClient integration.
- These packages are built on Polly. State accurately that direct Polly integration may be replaced while Polly remains transitive.
- Preserve end-to-end timeout budgets, strategy ordering, telemetry and failure semantics.
- Retry only safe/idempotent operations or those protected by idempotency/deduplication.
- Avoid stacked retries across client/service/gateway layers.
- Test transient/permanent failures, cancellation, exhaustion and circuit state using controlled failures.

VALIDATION
- Build affected projects.
- Run relevant unit, integration, contract, resilience and parity tests.
- Verify DI composition and application startup.
- Inspect package references and dependency graph.
- Search code/config/tests for stale references and old registrations.
- Verify no test-only dependency leaks into production.

DOCUMENTATION
Update the plan, package inventory, architecture ADR, dependency map, parity evidence, decision log and progress. Record what was removed, what remains transitively, what behaviour moved and why.

STOP CONDITIONS
Stop if a hidden behaviour cannot be preserved, replacement semantics are unclear, validation response changes, retry safety is unknown, a material package conflict exists, or tests cannot prove the migration.

FINAL RESPONSE
Provide a before/after dependency usage table, files changed, behaviours moved/preserved, tests and exact commands/results, dependency graph findings, risks and plan status. Do not start another dependency workstream automatically.
```

<!-- END DOCUMENT: prompts/05-dependency-modernization.md -->

---

<!-- BEGIN DOCUMENT: prompts/06-final-validation-and-dev-readiness.md -->

## Prompt 06 — Full regression, parity and DEV deployment readiness

> Run after every approved slice/workstream is complete. This prompt prepares readiness evidence; it does not authorize deployment.

```text
Act as a Principal .NET Architect, QA/Test Lead, DevSecOps reviewer and release-readiness assessor.

Read all modernization plans, current progress, responsibility matrix, API contract inventory, architecture decision, test baseline, parity matrices, performance reports, risk register and decision log.

The API remains on .NET 10. Do not upgrade the framework or broaden scope.

OBJECTIVE
Determine whether the modernized API is a credible candidate for DEV deployment and QA validation. This is an evidence review, not a declaration of guaranteed production safety.

PHASE 1 — Completeness and traceability
- Verify every in-scope responsibility and non-HTTP trigger is represented.
- Map each responsibility to the target code path/use case.
- Map every responsibility to tests and parity evidence.
- Verify every approved plan status and acceptance criterion.
- Identify any missing endpoint, background job, message handler, data side effect or downstream dependency.
- Confirm obsolete implementation paths are retired only when no longer required.

PHASE 2 — Build and test execution
Run the repository's actual validation matrix, not an assumed generic command:
1. Restore.
2. Build all relevant projects/configurations.
3. Unit tests.
4. Component/application tests.
5. Integration tests.
6. Contract tests.
7. Acceptance tests.
8. End-to-end tests.
9. Parity tests.
10. Security/authorization/tenant isolation tests.
11. Concurrency/idempotency tests.
12. Resilience/fault-injection tests.
13. Database migration/data compatibility tests, if applicable.
14. Performance/load/memory tests, where planned.
15. Static analysis, formatting and dependency/security checks used by CI.

For every command, record exact invocation, commit, SDK, environment, duration, passed/failed/skipped counts and artifact path. Distinguish pre-existing failures from new failures. Do not suppress failures or mark unexecuted tests as passing.

PHASE 3 — API and functional parity
For every endpoint/capability compare:
- Method/route/version/schema/status/headers/content type.
- Serialization, null, date/time, numeric precision and pagination semantics.
- AuthN/AuthZ and resource/tenant access.
- Validation errors and exception mapping.
- Business outcomes and resulting state.
- Database state, transactions, concurrency and idempotency.
- Events/messages and delivery-relevant metadata.
- Downstream calls, retries, timeouts and failure handling.
- Cache, audit and other relevant side effects.

Classify each difference. Unexplained differences block readiness. Confirm the parity harness did not cause duplicate real side effects.

PHASE 4 — Package and architecture review
- Search direct and transitive references.
- Confirm MediatR behaviours have been handled and no hidden pipeline remains.
- Confirm FluentValidation is removed or deliberately retained with reason.
- Confirm assertion replacement preserved test intent.
- Confirm direct Polly references and Microsoft.Extensions resilience use are understood; do not claim transitive Polly is absent if it remains.
- Review unnecessary abstractions, project cycles, duplicate code, DI scopes and test-only code.

PHASE 5 — Performance and resource review
Compare equivalent workload/environment against baseline. Include p50/p95/p99, throughput, CPU, memory/allocations, GC, error rate and database/downstream timings as available. Highlight regressions and measurement limitations. Do not infer gains from code style.

PHASE 6 — Security and operations review
Check:
- Authentication, claims/scopes, authorization and tenant isolation.
- Secrets/configuration references and environment differences.
- Logs/metrics/traces/correlation and sensitive-data redaction.
- Health/readiness and dependency checks.
- Timeouts, retries, circuit breakers and idempotency.
- Database schema/data compatibility and migration sequence.
- Message schema and producer/consumer rollout compatibility.
- Capacity/connection pool/thread pool limits where relevant.
- CI/CD artifact, runtime and app configuration compatibility.

PHASE 7 — DEV readiness pack
Create/update:
- `reports/final-validation-report.md`
- `reports/dev-readiness-report.md`
- `reports/test-evidence-index.md` (or equivalent)
- deployment checklist
- rollback plan
- QA scenario handover

The DEV readiness report must state:
- Candidate commit/build artifact.
- Implemented scope and remaining exclusions.
- Test matrix with exact results.
- API and functional parity status.
- Performance comparison and limitations.
- Package/dependency status.
- Database/message/configuration changes.
- Security and observability review.
- Known defects, residual risks and owners.
- DEV deployment prerequisites.
- Smoke tests and QA scenarios.
- Rollback trigger and procedure.
- Explicit go/no-go recommendation.

APPROVAL AND DEPLOYMENT RULE
Do not deploy, push, merge, create a release or modify environments. Only prepare readiness evidence. Human release owner approval is required for DEV deployment. If a deployment tool or environment is accessible, do not invoke it without explicit authorization.

FINAL RESPONSE
Give an evidence-based GO / CONDITIONAL GO / NO-GO recommendation for DEV. A GO means readiness criteria have been met for DEV deployment and QA testing; it does not mean production release is approved or guaranteed safe. List every unmet criterion and every unexecuted test.
```

<!-- END DOCUMENT: prompts/06-final-validation-and-dev-readiness.md -->

---

<!-- BEGIN DOCUMENT: prompts/07-independent-review.md -->

## Prompt 07 — Independent adversarial architecture and parity review

> Run as a separate review pass. Ideally use a fresh agent/session that did not implement the changes.

```text
Act as an independent Principal Architect, senior QA auditor and skeptical production-readiness reviewer. Your job is to find reasons this modernization may be incomplete or unsafe. Do not assume the implementation agent's report is correct.

Review the complete diff and all documentation under `docs/modernization/`. Verify claims against source code, tests, package references, configuration and CI outputs where accessible.

Do not make code changes during the first review pass.

REVIEW 1 — Scope completeness
- Is every API endpoint, background process, message handler and scheduled task accounted for?
- Are any upstream/downstream dependencies, data flows or side effects missing?
- Are responsibilities mapped to target implementations and tests?
- Are there undocumented behavioural assumptions?

REVIEW 2 — Contract and behaviour
- Did routes, verbs, versions, DTOs, nullability, serialization, status codes, errors, headers, pagination or auth change?
- Are changes approved and documented?
- Are there differences hidden by normalizers, broad JSON ignores or snapshot updates?
- Is functional parity proved beyond response JSON?

REVIEW 3 — Cross-cutting semantics
- Were all MediatR behaviours found and preserved or deliberately removed?
- Did validation order, async checks, error codes or response shape change?
- Were transaction boundaries, exception handling, caching, idempotency, logging, authorization and cancellation preserved?
- Are there retries that can duplicate writes or messages?
- Are timeouts nested or inconsistent?

REVIEW 4 — Architecture quality
- Does the target genuinely reduce coupling and complexity?
- Are there unnecessary projects, interfaces, generic repositories, factories or custom pipelines?
- Is any custom mediator merely recreating MediatR?
- Are responsibilities placed at appropriate boundaries?
- Are there cyclic dependencies, service locator patterns or incorrect DI lifetimes?
- Are test fakes impossible to select in production composition?

REVIEW 5 — Test strength
- Did tests get deleted, skipped, weakened or made less precise?
- Do tests use realistic provider semantics where needed?
- Are write/transaction/message side effects verified safely?
- Are concurrency, idempotency, auth, dependency failure and cancellation tested where required?
- Are tests deterministic and reproducible?
- Are failures accurately distinguished from baseline failures?

REVIEW 6 — Package and performance claims
- Are package removals real, including DI scanning/configuration and direct references?
- Are transitive dependencies described accurately?
- Are FluentValidation alternatives semantically equivalent?
- Is Microsoft.Extensions resilience correctly described as built on Polly?
- Are Span/Memory/records/pooling/ValueTask and other optimizations justified?
- Are performance claims backed by comparable measurements?

REVIEW 7 — Deployment readiness
- Can the artifact run with DEV configuration and supported runtime?
- Are schema changes backward-compatible and sequenced?
- Are message contracts compatible across rolling deployment?
- Are logs/metrics/traces/health checks useful?
- Is rollback technically possible with the data and message changes?
- Are there manual steps missing from the release checklist?

OUTPUT
Create `reports/independent-review.md` containing:
- Overall assessment and recommendation.
- Findings grouped as BLOCKER, HIGH, MEDIUM, LOW or OBSERVATION.
- For each finding: evidence (file/class/test/command), impact, likelihood, recommendation and required owner/approval.
- Missing or weak evidence.
- Tests that should be added or rerun.
- Exact changes required before DEV.
- Items that may safely be deferred with rationale.

Do not fix issues during this pass. First report findings. Do not declare GO if there is any unresolved blocker or unexplained parity difference.
```

<!-- END DOCUMENT: prompts/07-independent-review.md -->

---

<!-- BEGIN DOCUMENT: templates/plan-template.md -->

## PLAN-XXX — <Plan title>

## Status

NOT_STARTED | ASSESSMENT_COMPLETE | APPROVED | IN_PROGRESS | BLOCKED | IMPLEMENTED_AWAITING_VALIDATION | VALIDATION_FAILED | PARITY_FAILED | COMPLETE | DEFERRED | REJECTED

## Priority and owner

- Priority: P0 / P1 / P2 / P3
- Owner/reviewer:
- Target API/project:
- Related decision/ADR:

## 1. Goal

What measurable outcome should this plan achieve?

## 2. Business and technical rationale

Why is the change needed? What risk, cost or limitation does it address?

## 3. Existing responsibilities and behaviour to preserve

List the responsibility IDs, endpoints/triggers, business rules, public contracts, data behaviour, auth and side effects.

## 4. Current state and evidence

- Relevant project/files/types/methods:
- Existing tests:
- Baseline results:
- Known limitations:

## 5. Scope

### In scope

-

### Out of scope

-

## 6. Dependencies and prerequisites

- Upstream plans:
- Downstream plans:
- Required infrastructure/test data:
- Required approvals:

## 7. Target design

Describe the intended implementation, dependency direction and why it is simpler/safer.

## 8. Implementation steps

1.
2.
3.

## 9. Impact assessment

- API contract:
- Business behaviour:
- Data/schema/transactions:
- Messaging/events:
- External dependencies:
- Authentication/authorization:
- Resilience/timeouts:
- Observability:
- Performance/memory:
- CI/CD/deployment:

## 10. Test and parity plan

- Unit/component:
- Integration:
- Contract:
- Acceptance/E2E:
- Security/concurrency/resilience:
- Parity scenarios:
- Side-effect isolation:
- Performance validation:

## 11. Acceptance criteria

- [ ]
- [ ]
- [ ]

## 12. Risks and mitigations

| Risk | Impact | Likelihood | Mitigation | Owner |
|---|---|---|---|---|

## 13. Rollback

What is the safe rollback path? Consider code, data, schema, messages and configuration.

## 14. Validation commands and evidence

| Command/suite | Environment | Expected evidence | Actual result |
|---|---|---|---|

## 15. Definition of done

State objective conditions required before status can become COMPLETE.

## 16. Implementation notes and decision history

Record material deviations and approvals.

<!-- END DOCUMENT: templates/plan-template.md -->

---

<!-- BEGIN DOCUMENT: templates/responsibility-and-parity-matrix-template.md -->

## Responsibility and parity matrix

## A. Responsibility inventory

| Responsibility ID | Capability / trigger | Business purpose | Existing code path | Input | Output | Criticality | Target use case | Plan ID | Status |
|---|---|---|---|---|---|---|---|---|---|
| RESP-001 | | | | | | | | | |

## B. API contract matrix

| Contract ID | Method | Route/version | Request schema | Response schema | Status/error contract | Auth policy | Pagination/filter semantics | Consumer(s) | Contract test |
|---|---|---|---|---|---|---|---|---|---|
| API-001 | | | | | | | | | |

## C. Functional parity matrix

| Scenario ID | Responsibility | Scenario/input | Expected business outcome | Data/state change | Transaction result | Message/event side effect | Downstream call | Security expectation | Old result | New result | Difference classification | Evidence/status |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| PAR-001 | | | | | | | | | | | | |

Difference classification must be one of:

- MATCH
- REGRESSION
- APPROVED_CHANGE
- KNOWN_LEGACY_DEFECT
- NONDETERMINISTIC_WITH_APPROVED_NORMALIZATION
- UNRESOLVED

`UNRESOLVED` blocks plan completion. `APPROVED_CHANGE` must link to a decision/approval. A known defect must include an explicit disposition (preserve for compatibility, fix in this initiative, or defer).

## D. Test traceability matrix

| Responsibility ID | Unit | Component | Integration | Contract | Acceptance/E2E | Security | Resilience | Concurrency/idempotency | Parity evidence | Coverage gap/owner |
|---|---|---|---|---|---|---|---|---|---|---|
| RESP-001 | | | | | | | | | | |

<!-- END DOCUMENT: templates/responsibility-and-parity-matrix-template.md -->

---

<!-- BEGIN DOCUMENT: templates/risk-and-decision-log-template.md -->

## Risks and unknowns register

| ID | Type (risk/unknown/assumption) | Description | Affected responsibility/plan | Impact | Likelihood/confidence | Mitigation / investigation | Owner | Due/gate | Status |
|---|---|---|---|---|---|---|---|---|---|
| RISK-001 | | | | | | | | | |

## Architecture / implementation decision log

## DEC-XXX — <Decision title>

- Date:
- Status: PROPOSED / APPROVED / REJECTED / SUPERSEDED
- Decision owner/approver:
- Related plan(s):

### Context

What problem and constraints triggered the decision?

### Options considered

1. Option A — benefits, drawbacks, risks.
2. Option B — benefits, drawbacks, risks.
3. Option C — benefits, drawbacks, risks.

### Decision

What was selected?

### Rationale

Why is this the best fit for this API and current evidence?

### Consequences

- Positive:
- Negative/trade-offs:
- Follow-up work:

### Evidence and references

Link source files, tests, measurements and relevant documentation.

<!-- END DOCUMENT: templates/risk-and-decision-log-template.md -->

---

<!-- BEGIN DOCUMENT: templates/progress-tracker-template.md -->

## Modernization progress tracker

## Project summary

- API/project:
- Repository:
- Feature branch:
- Base commit:
- Target framework: net10.0
- Strategy ADR:
- Current stage/gate:
- Overall status:

## Plan status

| Order | Plan ID | Plan | Priority | Dependencies | Status | Validation evidence | Blocker/next action |
|---:|---|---|---|---|---|---|---|
| 1 | PLAN-001 | | P0 | | NOT_STARTED | | |

## Slice status

| Capability | Responsibility IDs | API operations/triggers | Real dependencies integrated | Unit | Integration | Contract | Acceptance | Parity | Status |
|---|---|---|---|---|---|---|---|---|---|
| | | | | | | | | | |

## Validation summary

- Last restore/build:
- Last complete test run:
- Pre-existing failures:
- New failures:
- Unexecuted suites:
- Latest parity report:
- Latest performance report:

## Current risks and decisions

- Open blockers:
- Approvals required:
- Residual risks:

## Next action

Describe the single next approved action and the gate it satisfies.

<!-- END DOCUMENT: templates/progress-tracker-template.md -->

---
