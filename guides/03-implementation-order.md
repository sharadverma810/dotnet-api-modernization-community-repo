# 03 — Recommended implementation order

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
