# 02 — Modernization stages and decision gates

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
